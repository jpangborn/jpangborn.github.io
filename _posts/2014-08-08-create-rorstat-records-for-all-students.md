---
layout: post
title: "Create RORSTAT Records for All Students"
modified: 2014-08-08
categories: "Banner Financial Aid"
excerpt: Simplify much of Banner Financial Aid processing by creating RORSTAT records for all potential students.
tags: [Banner, "Financial Aid", RORSTAT]
image:
  feature: night-sky.jpg
date: 2014-08-08T23:57:37-04:00
---

After setting up Banner Financial Aid, I quickly came to realize that writing population selections and rules was needlessly difficult. The problem was that we often want to work with populations of students that had no yet started the financial aid process for the year. These students had no record in the Financial Aid module, so writing population selections or rules often ignored these students altogether. After running into this issue a few times, I started looking for a solution. The simplest solution, to me, was to ensure that any student that I would want to work with in a year already had a record in the financial module, RORSTAT record.

The next step was to determine how to create the RORSTAT records and for whom. Let’s start with who to create RORSTAT records for. I thought about the widest group of students that we want to interact with throughout the year. At some point in the year, we interact with all active students in some way. But that doesn’t catch everyone. We also need to consider the incoming students. These students are not yet active students in Banner. So, our final population of students that we want to interact with is all active students and all incoming students who have been offered admission.

Next, I needed to determine how I was going to create the RORSTAT records. There are numerous ways that a RORSTAT record can be created in Banner. Here are some of the ways:

- Loaded an ISIR for the student.
- Assigned the student a tracking group.
- Gave the student a tracking requirement.
- Awarded the student a fund.

Obviously, there are parts of the year where we will not want to award and funds to students, and we have no controll over when or if the student files a FAFSA. Assigning the student a tracking group or giving them a tracking requirement are good methods beacuse both them are easy to setup in batch.

Since these two populations are somewhat different both in how we interact with them and in how they are stored in the database, I decided to deal with them separately.

## Incoming Students who have been Offered Admission

Since students are accepted on a rolling basis, the process of creating a RORSTAT for accepted students needs to be run on a daily basis. Because of this, I decided to batch post a tracking requirement. This is quicker than running students through the grouping process. (They will go through the grouping process later.) Below is the population selection, ADMIT_STU_NO_ADMIT_REQ:

{% highlight sql %}
SELECT saradap_pidm
FROM savapdc, saradap
WHERE saradap_term_code_entry = savapdc_term_code
  AND saradap_appl_no = savapdc_appl_no
  AND saradap_term_code_entry = &term_code
  AND savapdc_apdc_code LIKE 'A%'
  AND savapdc_apdc_code LIKE 'R%'
  AND savapdc_apdc_code IN ('GA','GP','GR')
  AND saradap_pidm NOT IN (*SUB *SUB_REQ_ADMIT)
{% endhighlight %}

This Population Selection selects all students with an Admissions Application for a term who have an approriate Application Decision code and do not have a satisfied ADMIT tracking requirement. The selection rules will need to be updated with the school’s appropriate application descision codes. This selection is looking for any Accepted student code, any Deposit Paid code, or any Graduate accepted or deposit paid codes.

Below are the rules for *SUB_REQ_ADMIT

{% highlight sql %}
SELECT rrrareq_pidm
FROM rrrareq
WHERE rrrareq_aidy_code = &aidy_code
  AND rrrareq_treq_code = 'ADMIT'
  AND rrrareq_sat_ind = 'Y'
{% endhighlight %}

After running the Population Selection, we use a Batch Posting rule to post the ADMIT requirement as Satisfied for each student. Here is the rule that should be entered on RORPOST.

| Field            | Value                  |
|:-----------------|:-----------------------|
| Category         | ADMIT                  |
| Creator ID       | DARYL                  |
| Application Code | FINAID                 |
| Selection ID     | ADMIT_STU_NO_ADMIT_REQ |
| User ID          | DARYL                  |
| Type Code        | R                      |
| Code to Post     | ADMIT                  |
| Status           | S                      |

This process will create a RORSTAT record for all accepted new students. Next we will create records for all active students.

## Continuing Students

Creating a RORSTAT record for continuing students by selecting all of the students who had a RORSTAT record in the previous year except those in some of our control groups. We exclude students in the following groups:

- NOAPST – No Admissions Application or Student Record
- NOACTV – Not an Active Student
- NOADMT – Not Offered Admission
- GRADUA – Graduated
- CANCEL – Cancelled their Admissions Application
- WTHDRW – Withdrawn
- REVIEW – Not in a Valid Group

This selects all of the students who might be back the next year. We run this process one time in mid December for the Aid Year that we will begin processing in January. Because the current aid year is not complete, we cannot be 100% accurate at this time. In particular, we do not attempt to exclude students who are expected to graduate at the end of current aid year. Our grouping in the next aid year and our Satisfactory Academic Progress process ensure that graduates are not awarded aid for the next year.

The poplulation selection, CREATE_NEW_YEAR_RORSTAT, for selecting these students is:

{% highlight sql %}
SELECT rorstat_pidm
FROM rorstat
WHERE rorstat_aidy_code = &pre_aidy_code
  AND rorstat_tgrp_code NOT IN ('NOAPST','NOACTV','NOADMT','GRADUA','CANCEL','WTHDRW','REVIEW')
{% endhighlight %}

We then run this selection of students through the groupding process, RORGRPS, with the following parameters.

| Parameter                     | Value                   |
|:------------------------------|:------------------------|
| Aid Year Code                 | 1112                    |
| Group Type Indicator          | T                       |
| Term Code                     |                         |
| Process Indicator             | B                       |
| Applicant ID                  |                         |
| Use All Applicants Indicator  | N                       |
| Application ID                | FINAID                  |
| Selection ID                  | CREATE_NEW_YEAR_RORSTAT |
| Creator ID                    | DARYL                   |
| User ID                       | DARYL                   |
| Period                        |                         |

Now you can be sure that the students that you want to work with will have a RORSTAT record. Population Selections and Rules suddenldy become much simpler to write. Stay tuned for a post about how we use tracking groups to further simplify writing population selections and rules.