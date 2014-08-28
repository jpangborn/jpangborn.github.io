---
layout: post
title: "Writing Web Snapshot Panes"
modified:
categories: "Banner Financial Aid"
excerpt: Learn how to write Web Snapshot Panes so that you can create custom layouts of Banner data.
tags: [banner, "financial aid", "web snapshot", sql]
comments: true
image:
  feature: berea-college-snow.jpg
date: 2013-03-25T00:00:00-04:00
---

Panes are the basic building blocks of Banner Financial Aid Web Snapshot. They are simply SQL queries, but there are some tricks to writing good Web Snapshot Panes. We’ll walk through creating a pane and then review some tricks for writing a variety of panes.

See my Introduction to Financial Aid Web Snapshot for background on getting started with Web Snapshot.

## Simple Panes

### Getting Started

The simplest panes are just basic SQL statements. Let’s start with a pane to display a student’s financial aid award for the aid year. In order to allow you select the correct information, Web Snapshot provides two dynamic variables that can be used in your SQL statements: Aid Year Code (:AIDY) and Personal ID Master (:PIDM). Your panes will most likely always use PIDM, but there are times when AIDY is not required. **TIP:** AIDY and PIDM must be upper case. Lower case will cause an error.

<!-- (gist: https://gist.github.com/jpangborn/5760334) -->

{% highlight sql %}
select rprawrd_fund_code as "Fund",
       rprawrd_awst_code as "Status",
       rprawrd_offer_amt as "Offered"
from rprawrd
where rprawrd_aidy_code = :AIDY
  and rprawrd_pidm = :PIDM
{% endhighlight %}

By default, Web Snapshot will format your query as a table with each column being one of the fields selected and each record having a separate row. The above SQL statement may produce the following table:

| Fund  | Status  | Offered  |
|:------|:--------|:---------|
| PELL  | ACPT    | 5000     |
| PROV  | ACPT    | 10000    |
| FDSL  | OFFR    | 5500     |
| WORK  | OFFR    | 7000     |

Notice that the name of each column was set to the name specified in the SQL statement. If no name is specified, the column name will be used. This leads to very ugly looking tables, so I recommend naming every column.

### Readable Data

We have nice column names, but some of the data isn’t in the friendliest format. Let’s modify the SQL to display the data in a more readable format. We will replace the Fund Code and Award Status Code with their associated descriptions and format the Offered Amount as currency.

<!-- (gist: https://gist.github.com/jpangborn/5760374) -->

{% highlight sql %}
select rfrbase_fund_title as "Fund",
       rtvawst_desc as "Status",
       to_char(nvl(rprawrd_offer_amt, 0), '$999,990') as "Offered"
from rprawrd
left join rfrbase
  on rprawrd_fund_code = rfrbase_fund_code
left join rtvawst
  on rprawrd_awst_code = rtvawst_code
where rprawrd_aidy_code = :AIDY
  and rprawrd_pidm = :PIDM;
{% endhighlight %}

Notice that we have joined to two other tables to get the descriptions that we needed. You will need to be familiar with data in Banner to write good Snaphsot panes. We also used a couple Oracle functions to format the Offered Amount. With these changes, our table is now more readable.

| Fund                         | Status    | Offered  |
|:-----------------------------|:----------|---------:|
| Federal Pell Grant           | Accepted  | $5,000   |
| Provost Scholarship          | Accepted  | $10,000  |
| Federal Sub. Stafford Loan   | Offered   | $5,500   |
| Federal Work-Study           | Offered   | $7,000   |

Now we have a nice looking table that displays the student’s award. Before we move on, let’s look at the details of adding this SQL statement to Web Snapshot as a pane. When adding a new snapshot pane, you will need to enter the following data:

-   Sequence Number
-   Pane Code
-   Description
-   Columns
-   SQL

The Pane Code is a four character code that identifies the pane. The Description is a text description that is used when building your layouts. SQL is self explanatory. We will discuss Sequence and Columns in the next section. For now, leave the Sequence at 1 and the Columns at 0.

### Sequences and Columns

The above table looks good and is readable, but wouldn’t it be nice to have a total at the bottom? This can be accomplished with Sequences. Web Snapshot allows you put up to three SQL statements together when creating a Pane. The sequence determines which order the SQL statements are displayed. Let’s add a grand total to the pane.

<!-- (gist: https://gist.github.com/jpangborn/5760390) -->

{% highlight sql %}
select to_char(sum(rprawrd_offer_amt), '$999,990') as "Total Offered:"
from rprawrd
where rprawrd_aidy_code = :AIDY
  and rprawrd_pidm = :PIDM
{% endhighlight %}

This generates the following table.

|               |
|---------------|
|Total Offered: |
|$27,000        |

This is a good start, but isn’t the best format for displaying the total. Here is where columns come in. The default value for Columns is 0. This creates a table with the format we have seen, where each field selected is a column. We can change this value to 1 to generate a table with all the fields selected in a column with field names to the left. Different Columns settings can be used, but I haven’t found a good use for anything other than 0 or 1. One possibility is using columns to generate a table that displays two rows per record. Below is the table generated with Columns set to 1.

|                |         |
|:---------------|--------:|
| Total Offered: | $27,000 |

I use a Columns setting of 1 often when I am selecting specific data for a student and I know that only one row will be returned. This is most useful in displaying summary data.

In order to enter this into Web Snapshot, you need to create another pane and enter the same code as the previous and set the sequence to 2 and the columns to 1. Putting it together and we have a pane that displays the following:

### Financial Aid Award

| Fund                         | Status    | Offered |
|:-----------------------------|:----------|--------:|
| Federal Pell Grant           | Accepted  | $5,000  |
| Provost Scholarship          | Accepted  | $10,000 |
| Federal Sub. Stafford Loan   | Offered   | $5,500  |
| Federal Work-Study           | Offered   | $7,000  |
|                         | Total Offered: | $27,000 |

## Advanced Tips and Tricks

Now that we have created a basic Snapshot pane, there are some advanced tips and tricks that you can use to create a number of different panes.

### Term-Based Panes

Though Web Snapshot only gives you access to two dynamic parameters, it is still possible select data only for a specific term or terms. Let’s modify our previous pane to include a term breakdown of the financial aid award. We will show the Fall and Sping terms whose codes for the 2012-2013 Aid Year are 201210 and 201220 respecitively. (Assume that the period codes match the term codes.)

<!-- (gist: https://gist.github.com/jpangborn/5760398) -->

{% highlight sql %}
select rfrbase_fund_title as "Fund",
       rtvawst_desc as "Status",
       to_char(nvl(f.rpratrm_offer_amt, 0), '$999,990') as "Fall",
       to_char(nvl(s.rpratrm_offer_amt, 0), '$999,990') as "Spring",
       to_char(nvl(rprawrd_offer_amt, 0), '$999,990') as "Total"
from rprawrd
inner join rfrbase
  on rprawrd_fund_code = rfrbase_fund_code
inner join rtvawst
  on rprawrd_awst_code = rtvawst_code
left join rpratrm f
  on rprawrd_aidy_code = f.rpratrm_aidy_code
  and rprawrd_pidm = f.rpratrm_pidm
  and rprawrd_fund_code = f.rpratrm_fund_code
  and f.rpratrm_period like '%10'
left join rpratrm s
  on rprawrd_aidy_code = s.rpratrm_aidy_code
  and rprawrd_pidm = s.rpratrm_pidm
  and rprawrd_fund_code = s.rpratrm_fund_code
  and s.rpratrm_period like '%20'
where rprawrd_aidy_code = :AIDY
  and rprawrd_pidm = :PIDM
{% endhighlight %}

The key here is the `rpratrm_period like '%10'` line. This allows us to limit results to the fall term for which ever aid year is specified. This method doesn’t work fully for any crossover terms, but there is a modification that resolves it. If you have a summer header that crosses over from the prior aid year, `rpratrm_period = '20' || (to_number(substr(rprawrd_aidy_code, 0, 2)) - 1) || '30'` will allow you to select data from just the summer header, while `rpratrm_period = '20' || substr(rprawrd_aidy_code, 0, 2) || '30'` will select data from just the summer trailer.

Below is the table generated for the above SQL.

| Fund                         | Status    | Fall    | Spring  | Total   |
|:-----------------------------|:----------|--------:|--------:|--------:|
| Federal Pell Grant           | Accepted  | $2,500  | $2,500  | $5,000  |
| Provost Scholarship          | Accepted  | $5,000  | $5,000  | $10,000 |
| Federal Sub. Stafford Loan   | Offered   | $2,750  | $2,750  | $5,500  |
| Federal Work-Study           | Offered   | $3,500  | $3,500  | $7,000  |

### Panes from Tables without Aid Year

The lack a term parameter becomes more of an issue when selecting data from a table that does not have an Aid Year code. This can be accomplished by joining with STVTERM. Below is an example selecting course registration.

<!-- (gist: https://gist.github.com/jpangborn/5760409) -->

{% highlight sql %}
select sfrstcr_term_code as "Term",
      ssbsect_subj_code || ' - ' || ssbsect_crse_numb as "Course",
      sfrstcr_crn as "CRN",
      ssbsect_ptrm_start_date || ' - ' || ssbsect_ptrm_end_date as "Course Dates",
      sfrstcr_gmod_code as "Grade Mode",
      sfrstcr_credit_hr as "Credit Hours",
      sfrstcr_bill_hr as "Bill Hours",
      sfrstcr_rsts_code as "Status"
from sfrstcr
inner join ssbsect
  on sfrstcr_term_code = ssbsect_term_code
  and sfrstcr_crn = ssbsect_crn
inner join stvterm
  on sfrstcr_term_code = stvterm_code
where stvterm_fa_proc_yr = :AIDY
  and sfrstcr_pidm = :PIDM
order by sfrstcr_term_code, "Course"
{% endhighlight %}

The key is the inner join to STVTERM. After this join, the statement will only select rows where the term code has an Financial Aid Processing Year that is equal to the Aid Year specified.

### Panes for Different Aid Years

If want to display data from a different aid year next to data from the current aid year, this trick will enable that.

<!-- (gist: https://gist.github.com/jpangborn/5760418) -->

{% highlight sql %}
select to_char(nvl(rnvand0_budget_amount, 0), '$999,990') as "Budget",
       to_char(nvl(rnvand0_efc_amt, 0), '$999,990') as "EFC",
       to_char(nvl(rnvand0_gross_need, 0), '$999,990') as "Gross Need",
       to_char(nvl(rnvand0_offer_amt, 0), '$999,990') as "Total Aid",
       to_char(nvl(rnvand0_unmet_need, 0), '$999,990') as "Unmet Need"
from rnvand0
where rnvand0_aidy_code = :AIDY - 101
  and rnvand0_pidm = :PIDM
{% endhighlight %}

This pane will show the student’s need calcuation from the previous aid year. The key here is the `rnvand0_aidy_code = :AIDY - 101` line. If the current aid year is 1314, it will select data from 1213.

## More Examples

I recently released most of my [Web Snapshot Panes](https://github.com/jpangborn/banr-web-snapshot-panes) on Github. It include panes for selecting a variety of data including:

-   Aid Year Award
-   Aid Year Holds
-   Aid Year Overview
-   Disbursement Errors
-   Financial Aid Award by Term
-   GPA Information
-   Need Calculation
-   Student Bill for Fall, Spring, and Summer Terms
-   Student Budget
-   Student Course Registration
-   Student Holds
-   Ten Most Recent Letters
-   Tracking Requirements

All the information needed to setup the panes is included. You will also see other SQL features used, such as ordering results, limiting results, and calling functions.

Hopefully this helps you get started with developing Web Snapshot Panes. If you have any useful panes that you use, I would like to here about them. You may either issue a Pull Request on Github or post a comment here. I would also like to here about ideas for panes. If anyone has a pane ides that they haven’t been able to implement, I would be willing to give it a try and post the result.