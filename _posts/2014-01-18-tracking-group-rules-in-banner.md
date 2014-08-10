---
layout: post
title: "Tracking Group Rules in Banner"
modified:
categories: "Banner Financial Aid"
excerpt: Learn how to write tracking group rules to implement the new verification groups in Banner.
tags: [banner, "financial aid", verification, tracking]
image:
  feature:
date: 2014-01-18T00:00:00-05:00
---

<section id="table-of-contents" class="toc">
  <header>
    <h3>Overview</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->

A year later, I am now getting to posting a follow-up to my article about Processing New Verification Groups in Banner. The previous article describes the tracking group setup. Here, I will go through the SQL rules needed to implement that setup.

## Getting Started

Before writing any rules, we need to know what data will be used in determining to which group a student will be assigned. At the lowest level, we will need to look at two records: the student record (SGBSTDN) or the admissions application (SARADAP). New students won’t have a student record until later in the aid year, and continuing students won’t have an admissions record. Sometimes a student will have both, such as a student who withdrew previously and is reapplying for admission. Now we are left with determining when to use each record. For our rules, we decided that priority would be given to the admissions application. So if a student has an admissions application for one of the terms in the aid year, the admissions application will be used. Otherwise, the student record will be used. It is possible that a student wouldn’t have either record. In order handle all this decision-making and allow us to write simpler rules, we will create a view.

## RZVTGRP

In short, a view is simply a SQL statement that we save to the database. We can then access the results of that statement as if it is a table. The view will handle the decision making of which record to pull the data from. Much of the data elements in each record is the same. When creating the view, we will start with RORSTAT as the base table. We create RORSTAT records for all students, so that we know a RORSTAT record exists and we set tracking groups to run for all RORSTAT records, so it makes a great base table. From there, we are going to conditionally pull fields from the appropriate record to fill in the other columns. Here is the list of columns that we want to gather.

- PIDM
- Aid Year Code
- Module (Indicates whether admissions application or student record used.)
- Term Code
- Level Code
- Student Type
- Major Code
- EGOL Code
- Admit Code
- Student Status
- Decision Code
- Class Code

Below is the code of RZVTGRP:

<!-- {% gist jpangborn/8487191 %} -->

{% highlight sql %}
CREATE OR REPLACE VIEW baninst1.rzvtgrp AS
  SELECT rorstat_pidm AS rzvtgrp_pidm,
         rorstat_aidy_code AS rzvtgrp_aidy_code,
         CASE
           WHEN adm_rec IS NOT NULL
             THEN 'A'
           WHEN stu_rec IS NOT NULL
             THEN 'S'
           ELSE NULL
         END AS rzvtgrp_module,
         CASE
           WHEN adm_rec IS NOT NULL
             THEN saradap_term_code_entry
           WHEN stu_rec IS NOT NULL
             THEN sgbstdn_term_code_eff
           ELSE NULL
         END AS rzvtgrp_term_code,
         CASE
           WHEN adm_rec IS NOT NULL
             THEN saradap_levl_code
           WHEN stu_rec IS NOT NULL
             THEN sgbstdn_levl_code
           ELSE NULL
         END AS rzvtgrp_levl_code,
         CASE
           WHEN adm_rec IS NOT NULL
             THEN saradap_styp_code
           WHEN stu_rec IS NOT NULL
             THEN sgbstdn_styp_code
           ELSE NULL
         END AS rzvtgrp_styp_code,
         CASE
           WHEN adm_rec IS NOT NULL
             THEN saradap_majr_code_1
           WHEN stu_rec IS NOT NULL
             THEN sgbstdn_majr_code_1
           ELSE NULL
         END AS rzvtgrp_majr_code,
         CASE
           WHEN adm_rec IS NOT NULL
             THEN saradap_egol_code
           WHEN stu_rec IS NOT NULL
             THEN sgbstdn_egol_code
           ELSE NULL
         END AS rzvtgrp_egol_code,
         CASE
           WHEN adm_rec IS NOT NULL
             THEN saradap_admt_code
           ELSE NULL
         END AS rzvtgrp_admt_code,
         CASE
           WHEN stu_rec IS NOT NULL
             THEN sgbstdn_stst_code
           ELSE NULL
         END AS rzvtgrp_stst_code,
         CASE
           WHEN adm_rec IS NOT NULL
             THEN rzkutil.f_apdc_code(rorstat_pidm,
                                      saradap_term_code_entry,
                                      saradap_appl_no)
           ELSE NULL
         END AS rzvtgrp_apdc_code,
         CASE
           WHEN stu_rec IS NOT NULL
             THEN f_class_calc_fnc(pidm, sgbstdn_levl_code, term_code)
           ELSE NULL
         END AS rzvtgrp_clas_code
  FROM (SELECT rorstat_pidm,
               rorstat_aidy_code,
               rzkrecs.f_aidy_stu_rec(rorstat_pidm, rorstat_aidy_code) AS stu_rec,
               rzkrecs.f_aidy_adm_rec(rorstat_pidm, rorstat_aidy_code) AS adm_rec,
               rzktuil.f_aidy_term_code(rorstat_aidy_code) AS term_code
        FROM rorstat) r
  LEFT JOIN sgbstdn
    ON sgbstdn.ROWID = r.stu_rec
  LEFT JOIN saradap
    ON saradap.ROWID = r.adm_rec;

COMMENT ON TABLE BANINST1.rzvtgrp IS 'Tracking Grouping View shows Student or Admissions Record.';

CREATE OR REPLACE PUBLIC SYNONYM rzvtgrp FOR BANINST1.rzvtgrp;
{% endhighlight %}

Let’s break down this view. At the high level, we will select a number of columns from a subselect statement. In the subselect, we are making two function calls that return the Row ID of the needed SGBSTDN and SARADAP records. We are then using those Row IDs to outer join those tables to RORSTAT. We then use CASE statements to choose the appropriate record to pull the data from.

Here are the two functions that return Row IDs.

<!-- {% gist jpangborn/8487348 %} -->

{% highlight sql %}
-- Retreive the ROWID for the Most Recent Student Record for a Period.
FUNCTION f_aidy_stu_rec(pidm_in IN NUMBER,
                        aidy_in IN VARCHAR2)
                        RETURN ROWID IS

  r_id ROWID;

  CURSOR sgbstdn_list IS
    SELECT sgbstdn.rowid
    FROM sgbstdn
    WHERE sgbstdn_pidm = pidm_in
      AND sgbstdn_term_code_eff <= (SELECT max(stvterm_code)
                                    FROM stvterm
                                    WHERE stvterm_fa_proc_yr = aidy_in)
    ORDER BY sgbstdn_term_code_eff DESC;

BEGIN
  BEGIN
    OPEN sgbstdn_list;
    FETCH sgbstdn_list INTO r_id;

    IF sgbstdn_list%NOTFOUND THEN
      r_id:= NULL;
    END IF;

    CLOSE sgbstdn_list;
  END;

  RETURN r_id;

END f_aidy_stu_rec;

-- Retreive the Most Recent Admissions Record for the Aid Year
FUNCTION f_aidy_adm_rec(pidm_in in NUMBER,
                        aidy_in in VARCHAR2)
                        RETURN ROWID IS

  r_id ROWID;

  CURSOR saradap_list IS
    SELECT saradap.rowid
      FROM saradap
    LEFT JOIN stvterm
      ON saradap_term_code_entry = stvterm_code
    WHERE saradap_pidm = pidm_in
      AND stvterm_fa_proc_yr = aidy_in
    ORDER BY saradap_term_code_entry DESC,
             saradap_appl_no DESC;

BEGIN
  BEGIN
    OPEN saradap_list;
    FETCH saradap_list INTO r_id;

    IF saradap_list%NOTFOUND THEN
      r_id := NULL;
    END IF;

    CLOSE saradap_list;
  END;

  RETURN r_id;
END f_aidy_adm_rec;
{% endhighlight %}

For RZVTGRP, the CASE statements are pretty straightforward. If an admissions application exists, we use data from it. Otherwise, if a student record exists, we use it. Finally, if neither exists the columns will be be NULL.

Now that we have the base record dealt with, we can begin to build rules for all the groups that were defined. we will walk through a number of the rules.

## Rules

We will start with the control groups that gather students that we will not be processing.

### XNAPST – No Admission Application or Student Record

This is a pretty straight-forward rule that uses the module column of the RZVTGRP view. Notice the `:PIDM` and `:AIDY` in the view. These are substitution variables allowed by Banner to make writing the rules easier. We don’t need to hard code the student’s internal ID or the Aid Year. Because of this, the same rule can be used across students and aid years.

<!-- {% gist jpangborn/8494806 %} -->

{% highlight sql %}
-- XNAPST - No Admis App/Stu Record

SELECT DISTINCT(rzvtgrp_pidm)
FROM rzvtgrp
WHERE rzvtgrp_aidy_code = :AIDY
  AND rzvtgrp_pidm = :PIDM
  AND rzvtgrp_module IS NULL;
{% endhighlight %}

### XNADMT – Students who are not Admitted

Here is a rule that doesn’t use the RZVTGRP view. It could have been written using the view, but didn’t need to be. This group only contains students who have an admissions application, and since we give priority to admissions application, we can write the rule directly against the admissions tables. Any students without an admissions application will move on to process the next rule.

<!-- {% gist jpangborn/8494961 %} -->

{% highlight sql %}
-- XNADMT - Not Admitted

SELECT DISTINCT(saradap_pidm)
FROM savapdc, saradap
WHERE savapdc_term_code = saradap_term_code_entry
  AND savapdc_appl_no = saradap_appl_no
  AND savapdc_pidm = saradap_pidm
  AND saradap_term_code_entry =
    (SELECT MAX(saradap_term_code_entry)
    FROM saradap, stvterm
    WHERE saradap_term_code_entry = stvterm_code
      AND stvterm_fa_proc_yr = :AIDY
      AND saradap_pidm = :PIDM)
  AND ((savapdc_apdc_code NOT LIKE 'A%'
    AND savapdc_apdc_code NOT LIKE 'X%'
    AND savapdc_apdc_code NOT LIKE 'R%'
    AND savapdc_apdc_code NOT IN ('GA', 'GX', 'GP', 'GR', 'GC', 'GV'))
    OR savapdc_apdc_code IS NULL)
  AND saradap_pidm = :PIDM
{% endhighlight %}

### XNACTV - Students who are not Active

Another very simple rule because of our view. Selects any student whose data comes from the student record and who has an inactive student status. Your codes may be different.

<!-- {% gist jpangborn/8494887 %} -->

{% highlight sql %}
-- XNACTV - Not Active

SELECT DISTINCT(rzvtgrp_pidm)
FROM rzvtgrp
WHERE rzvtgrp_aidy_code = :AIDY
  AND rzvtgrp_pidm = :PIDM
  AND rzvtgrp_module = 'S'
  AND rzvtgrp_admt_code = 'IS'
{% endhighlight %}

### XWTHDN - Withdrawn

Another simple rule selecting students who have a withdrawn student status.

<!-- {% gist jpangborn/8495040 %} -->

{% highlight sql %}
-- XWTHDN - Withdrawn

SELECT DISTINCT(rzvtgrp_pidm)
FROM rzvtgrp
WHERE rzvtgrp_aidy_code = :AIDY
  AND rzvtgrp_pidm = :PIDM
  AND rzvtgrp_module = 'S'
  AND rzvtgrp_admt_code LIKE 'W%'
{% endhighlight %}

### XCNCLD – Cancelled

This rule is very similar to the XADMT rule, except it looks for students with admissions application decision codes that indicate the student cancelled.

<!-- {% gist jpangborn/8495087 %} -->

{% highlight sql %}
-- XCNCLD - Cancelled

SELECT DISTINCT(saradap_pidm)
FROM savapdc, saradap
WHERE savapdc_term_code = saradap_term_code_entry
  AND savapdc_appl_no = saradap_appl_no
  AND savapdc_pidm = saradap_pidm
  AND saradap_term_code_entry =
    (SELECT MAX(saradap_term_code_entry)
    FROM saradap, stvterm
    WHERE saradap_term_code_entry = stvterm_code
      AND stvterm_fa_proc_yr = :AIDY
      AND saradap_pidm = :PIDM)
  AND (savapdc_apdc_code LIKE 'X%'
    OR savapdc_apdc_code = 'GX')
  AND saradap_pidm = :PIDM
{% endhighlight %}

That ends the non-processing groups. For the remained of the groups I will focus on the undergraduate versions. The graduate version are very similar for the most part. I may also skip some Undergrad rules that use similar concepts to previous rules.

### UNODEG – Undergrad not seeking a degree

A pretty simple rule that uses RZVTGRP. Notice that this is the first rule using RZVTGRP that doesn’t care which module the data comes from. It allows the view to provide the correct data and just checks the values. The UDLENR and UVISIT rules are very similar to this rule.

<!-- {% gist jpangborn/8495177 %} -->

{% highlight sql %}
-- UNODEG - UG - Non-Degree Seeking

SELECT DISTINCT(rzvtgrp_pidm)
FROM rzvtgrp
WHERE rzvtgrp_aidy_code = :AIDY
  AND rzvtgrp_pidm = :PIDM
  AND rzvtgrp_levl_code = 'UG'
  AND rzvtgrp_styp_code = 'S'
{% endhighlight %}

### UNOFIL – Undergrad did not file a FAFSA

The following rule selects students who have not filed a FAFSA. We are looking at RORSTAT to determine if the student has an Application Received Date. Notice the final lines of this rule. They cover a unique situation where an accelerated graduate degree program admits students who will be undergraduate for a time. In most cases, you could look just at the level code.

<!-- {% gist jpangborn/8495412 %} -->

{% highlight sql %}
-- UNOFIL - UG - No FAFSA on File

SELECT DISTINCT(rorstat_pidm)
FROM rorstat
INNER JOIN rzvtgrp
  ON rorstat_pidm = rzvtgrp_pidm
  AND rorstat_aidy_code = rzvtgrp_aidy_code
WHERE rorstat_aidy_code = :AIDY
  AND rorstat_pidm = :PIDM
  AND rorstat_appl_rcvd_date IS NULL
  AND ((rzvtgrp_module = 'S'
    AND rzvtgrp_levl_code = 'UG')
  OR (rzvtgrp_module = 'A'
    AND (rzvtgrp_levl_code = 'UG'
    OR (rzvtgrp_levl_code = 'GR'
      AND rzvtgrp_styp_code = 'B'))))
{% endhighlight %}

### UGRJISR – Undergrad with a rejected ISIR

Here we look at FAFSA data to determine whether the ISIR has been rejected. We are also using RZVTGRP to get the correct level code for the student.

<!-- {% gist jpangborn/8495617 %} -->

{% highlight sql %}
-- URJISR - UG - Rejected ISIR

SELECT DISTINCT(rcrapp1_pidm)
FROM rcrapp1
INNER JOIN rcrapp3
  ON rcrapp1_aidy_code = rcrapp3_aidy_code
  AND rcrapp1_pidm = rcrapp3_pidm
  AND rcrapp1_infc_code = rcrapp3_infc_code
  AND rcrapp1_seq_no = rcrapp3_seq_no
INNER JOIN rzvtgrp
  ON rcrapp1_aidy_code = rzvtgrp_aidy_code
  AND rcrapp1_pidm = rzvtgrp_pidm
WHERE rcrapp1_aidy_code = :AIDY
  AND rcrapp1_pidm = :PIDM
  AND rcrapp1_curr_rec_ind = 'Y'
  AND rcrapp3_offl_unoffl_ind = '2'
  AND ((rzvtgrp_module = 'S'
    AND rzvtgrp_levl_code = 'UG')
  OR (rzvtgrp_module = 'A'
    AND (rzvtgrp_levl_code = 'UG'
      OR (rzvtgrp_levl_code = 'GR'
        AND rzvtgrp_styp_code = 'B'))))
{% endhighlight %}

### UDNOVR – Undergrad Dependent Not Selected for Verification

Starting here, the groups begin to break down Dependent and Independent students. I will show the rules for the dependent groups. The rules for the independent groups are very similar. Here we are again looking at FAFSA data, this time to determine dependency status and whether the student is selected for verification. Again using RZVTGRP to determine level code. UINOVR is very similar except looking for an ‘I’ in `rcrapp2_model_cde`. GINOVR is also similar except for different level code criteria.

<!-- {% gist jpangborn/8495941 %} -->

{% highlight sql %}
-- UDNOVR - UG - Dep - No Verify

SELECT DISTINCT(rcrapp1_pidm)
FROM rcrapp1
INNER JOIN rcrapp2
  ON rcrapp1_aidy_code = rcrapp2_aidy_code
  AND rcrapp1_pidm = rcrapp2_pidm
  AND rcrapp1_infc_code = rcrapp2_infc_code
  AND rcrapp1_seq_no = rcrapp2_seq_no
INNER JOIN rzvtgrp
  ON rcrapp1_aidy_code = rzvtgrp_aidy_code
  AND rcrapp1_pidm = rzvtgrp_pidm
WHERE rcrapp1_aidy_code = :AIDY
  AND rcrapp1_pidm = :PIDM
  AND rcrapp1_curr_rec_ind = 'Y'
  AND rcrapp1_verification_msg = '2'
  AND rcrapp2_model_cde = 'D'
  AND ((rzvtgrp_module = 'S'
    AND rzvtgrp_levl_code = 'UG')
  OR (rzvtgrp_module = 'A'
    AND (rzvtgrp_levl_code = 'UG'
      OR (rzvtgrp_levl_code = 'GR'
        AND rzvtgrp_styp_code = 'B'))))
{% endhighlight %}

Now we have come to main verification groups. Verification groups are broken down by level, dependency status, and tax filing status. We will look at the first V1 rule and then describe how the other V1 groups are different. The other verification groups are very similar.

### UDV1TT – Undergrad Dependent Selected for V1 Verification

Like the previous rule, we will look at FAFSA data and RZVTGRP for dependency status and level code. We will also look at the FAFSA data for the verification information and the tax filing info. We find whether a student is selected for verification in rcrapp1_verification_msg and which verification group in `rcrapp1_verification_prty`. In order to determine tax filing status, we need to look at two separate fields, one for the student and one for the parents. For the student, we look at `rcrapp4_tx_ret_filed_ind`; for the parents, we look at `rcrapp4_par_tx_ret_filed_ind`. There are a number of possible statuses for these filed. If the student or parent indicated that they filed a tax return, will file a tax return, or didn’t answer we group them with filed taxes. Values of ’1′, ’2′, or NULL cover these situations.

<!-- {% gist jpangborn/8496233 %} -->

{% highlight sql %}
-- UDV1TT - UG - Dep - V1 - Stu: T Par: T

SELECT DISTINCT(rcrapp1_pidm)
FROM rcrapp1
INNER JOIN rcrapp2
  ON rcrapp1_aidy_code = rcrapp2_aidy_code
  AND rcrapp1_pidm = rcrapp2_pidm
  AND rcrapp1_infc_code = rcrapp2_infc_code
  AND rcrapp1_seq_no = rcrapp2_seq_no
INNER JOIN rcrapp4
  ON rcrapp1_aidy_code = rcrapp4_aidy_code
  AND rcrapp1_pidm = rcrapp4_pidm
  AND rcrapp1_infc_code = rcrapp4_infc_code
  AND rcrapp1_seq_no = rcrapp4_seq_no
INNER JOIN rzvtgrp
  ON rcrapp1_aidy_code = rzvtgrp_aidy_code
  AND rcrapp1_pidm = rzvtgrp_pidm
WHERE rcrapp1_aidy_code = :AIDY
  AND rcrapp1_pidm = :PIDM
  AND rcrapp1_curr_rec_ind = 'Y'
  AND rcrapp1_verification_msg = '1'
  AND rcrapp1_verification_prty = 'V1'
  AND rcrapp2_model_cde = 'D'
  AND (rcrapp4_tx_ret_filed_ind IN ('1','2')
    OR rcrapp4_tx_ret_filed_ind IS NULL)
  AND (rcrapp4_par_tx_ret_filed_ind IN ('1','2')
    OR rcrapp4_par_tx_ret_filed_ind IS NULL)
  AND ((rzvtgrp_module = 'S'
      AND rzvtgrp_levl_code = 'UG')
    OR (rzvtgrp_module = 'A'
      AND (rzvtgrp_levl_code = 'UG'
        OR (rzvtgrp_levl_code = 'GR'
          AND rzvtgrp_styp_code = 'B'))))
{% endhighlight %}

To cover the other possibilities for tax filing status, we will change the criteria for either rcrapp4_tx_ret_filed_ind or rcrapp4_par_tx_ret_filed_ind.

### UDV1TN - Undergrad Dependent Selected for V1 Verification

This group differs from the previous rule in that the parents indicated that they would not file a tax return. So we change the criteria for `rcrapp4_par_tx_ret_filed_ind` from ’1′, ’2′, or NULL to ’3′

<!-- {% gist jpangborn/8496546 %} -->

{% highlight sql %}
-- UDV1TN - UG - Dep - V1 - Stu: T Par: N

SELECT DISTINCT(rcrapp1_pidm)
FROM rcrapp1
INNER JOIN rcrapp2
  ON rcrapp1_aidy_code = rcrapp2_aidy_code
  AND rcrapp1_pidm = rcrapp2_pidm
  AND rcrapp1_infc_code = rcrapp2_infc_code
  AND rcrapp1_seq_no = rcrapp2_seq_no
INNER JOIN rcrapp4
  ON rcrapp1_aidy_code = rcrapp4_aidy_code
  AND rcrapp1_pidm = rcrapp4_pidm
  AND rcrapp1_infc_code = rcrapp4_infc_code
  AND rcrapp1_seq_no = rcrapp4_seq_no
INNER JOIN rzvtgrp
  ON rcrapp1_aidy_code = rzvtgrp_aidy_code
  AND rcrapp1_pidm = rzvtgrp_pidm
WHERE rcrapp1_aidy_code = :AIDY
  AND rcrapp1_pidm = :PIDM
  AND rcrapp1_curr_rec_ind = 'Y'
  AND rcrapp1_verification_msg = '1'
  AND rcrapp1_verification_prty = 'V1'
  AND rcrapp2_model_cde = 'D'
  AND (rcrapp4_tx_ret_filed_ind IN ('1','2')
    OR rcrapp4_tx_ret_filed_ind IS NULL)
  AND rcrapp4_par_tx_ret_filed_ind = '3'
  AND ((rzvtgrp_module = 'S'
      AND rzvtgrp_levl_code = 'UG')
    OR (rzvtgrp_module = 'A'
      AND (rzvtgrp_levl_code = 'UG'
        OR (rzvtgrp_levl_code = 'GR'
          AND rzvtgrp_styp_code = 'B'))))
{% endhighlight %}

### UDV1NT – Undergrad Dependent Selected for V1 Verification

Here the student indicated they would not file, while the parents did or will. So we change the criteria for `rcrapp4_tx_ret_filed_ind` from ’1′, ’2′, or NULL to ’3′ and `rcrapp4_par_tx_ret_filed_ind` goes back to the original values.

<!-- {% gist jpangborn/8496634 %} -->

{% highlight sql %}
-- UDV1NT - UG - Dep - V1 - Stu: N Par: T

SELECT DISTINCT(rcrapp1_pidm)
FROM rcrapp1
INNER JOIN rcrapp2
  ON rcrapp1_aidy_code = rcrapp2_aidy_code
  AND rcrapp1_pidm = rcrapp2_pidm
  AND rcrapp1_infc_code = rcrapp2_infc_code
  AND rcrapp1_seq_no = rcrapp2_seq_no
INNER JOIN rcrapp4
  ON rcrapp1_aidy_code = rcrapp4_aidy_code
  AND rcrapp1_pidm = rcrapp4_pidm
  AND rcrapp1_infc_code = rcrapp4_infc_code
  AND rcrapp1_seq_no = rcrapp4_seq_no
INNER JOIN rzvtgrp
  ON rcrapp1_aidy_code = rzvtgrp_aidy_code
  AND rcrapp1_pidm = rzvtgrp_pidm
WHERE rcrapp1_aidy_code = :AIDY
  AND rcrapp1_pidm = :PIDM
  AND rcrapp1_curr_rec_ind = 'Y'
  AND rcrapp1_verification_msg = '1'
  AND rcrapp1_verification_prty = 'V1'
  AND rcrapp2_model_cde = 'D'
  AND rcrapp4_tx_ret_filed_ind = '3'
  AND (rcrapp4_par_tx_ret_filed_ind IN ('1','2')
    OR rcrapp4_par_tx_ret_filed_ind IS NULL)
  AND ((rzvtgrp_module = 'S'
      AND rzvtgrp_levl_code = 'UG')
    OR (rzvtgrp_module = 'A'
      AND (rzvtgrp_levl_code = 'UG'
        OR (rzvtgrp_levl_code = 'GR'
          AND rzvtgrp_styp_code = 'B'))))
{% endhighlight %}

### UDV1NN – Undergrad Dependent Selected for V1 Verification

Here neither student nor parent will file a tax return criteria on both tax return filed indicator fields will need to be ’3′.

<!-- {% gist jpangborn/8496747 %} -->

{% highlight sql %}
-- UDV1NN- UG - Dep - V1 - Stu: N Par: N

SELECT DISTINCT(rcrapp1_pidm)
FROM rcrapp1
INNER JOIN rcrapp2
  ON rcrapp1_aidy_code = rcrapp2_aidy_code
  AND rcrapp1_pidm = rcrapp2_pidm
  AND rcrapp1_infc_code = rcrapp2_infc_code
  AND rcrapp1_seq_no = rcrapp2_seq_no
INNER JOIN rcrapp4
  ON rcrapp1_aidy_code = rcrapp4_aidy_code
  AND rcrapp1_pidm = rcrapp4_pidm
  AND rcrapp1_infc_code = rcrapp4_infc_code
  AND rcrapp1_seq_no = rcrapp4_seq_no
INNER JOIN rzvtgrp
  ON rcrapp1_aidy_code = rzvtgrp_aidy_code
  AND rcrapp1_pidm = rzvtgrp_pidm
WHERE rcrapp1_aidy_code = :AIDY
  AND rcrapp1_pidm = :PIDM
  AND rcrapp1_curr_rec_ind = 'Y'
  AND rcrapp1_verification_msg = '1'
  AND rcrapp1_verification_prty = 'V1'
  AND rcrapp2_model_cde = 'D'
  AND rcrapp4_tx_ret_filed_ind = '3'
  AND rcrapp4_par_tx_ret_filed_ind ='3'
  AND ((rzvtgrp_module = 'S'
      AND rzvtgrp_levl_code = 'UG')
    OR (rzvtgrp_module = 'A'
      AND (rzvtgrp_levl_code = 'UG'
        OR (rzvtgrp_levl_code = 'GR'
          AND rzvtgrp_styp_code = 'B'))))
{% endhighlight %}

All of the rest of the rules are variations on the above. The Independent variations of the V1 groups will only need to reference whether the student filed taxes or not, so there is only two variations.  The V2, V3, and V4 groups don’t have any of the taxes filed indicator criteria. The V5 group rules are identical to the V1 rules except for the verification priority criteria.

I will be packaging all of the rules and the needed views and packages and hosting them on Github.