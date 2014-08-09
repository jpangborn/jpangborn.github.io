---
layout: post
title: "Processing New Verification Groups in Banner"
modified:
categories: "Banner Financial Aid"
excerpt: Learn how I updated Banner to process the new verification groups in Banner.
tags: [banner, "financial aid", verification]
image:
  feature: berea-college-bushy-fork.jpg
date: 2013-02-15T00:00:00-05:00
---

The new verification groups and associated regulations for 2013-2014 have caused us to rethink how we process verification in Banner. I’ll detail our thinking and how we setup Tracking Groups to handle the new verification process.

If you haven’t already, read my introduction to the [verification changes for 2013-2014]({% post_url 2013-01-15-2013-2014-verification-changes %}) before continuing.

## Previous Setup

Our previous tracking group setup regarding verification was to have 6 groups:

-   Graduate – Not Selected for Verification
-   Graduate – Selected for Verification
-   Undergraduate – Dependent Not Selected for Verification
-   Undergraduate – Dependent Student Selected for Verification
-   Undergraduate – Independent Student Not Selected for Verification
-   Undergraduate – Independent Student Selected for Verification

In previous years, the above groups handled assigning the following verification requirements:

### Graduate – Selected for Verification

-   Students Federal Tax Return
-   Verification Worksheet
-   Internal Verify Requirement
-   The Internal Verify Requirement was our indication that all documents have been reviewed and verification is complete.

### Undergraduate – Independent Student Selected for Verification

-   Student’s Federal Tax Return
-   Verification Worksheet
-   Internal Verify Requirement

### Undergraduate – Dependent Student Selected for Verification

-   Student’s Federal Tax Return
-   Parents’ Federal Tax Return
-   Verification Worksheet
-   Internal Verify Requirement

This setup served us pretty well up until the 2012-2013 Aid Year. The changes concerning acceptable tax documents and the IRS Data Retrieval Tool required some changes. To accommodate these changes, we switched the Federal Tax Return requirements to Federal Tax Data requirements and then implemented some batch posting to satisfy the requirement with a unique status if the student or parents completed the IRS Data Retrieval without changing any data.

In past years we also manually handled students or parents who did not file taxes, adjusting their requirements to W2s. With the above changes, we also built a batch posting process to waive the new Tax Data requirement and post W2 requirements if the student or parents indicated that they did not intend to file a tax return.

## New Setup

### Tracking Groups

As we started thinking about how we wanted to handle the new verification groups, we decided it was time to reevaluate the entire process. It would have been relatively easy to just break out the selected for verification groups for the new verification groups or to use batch posting to post the various requirements. I immediately leaned toward groups rather than batch posting for one primary reason. Students can easily change groups. A student in one group can move to another group and have their requirements change. This is significantly more difficult to accomplish using batch posting. So we began discussing the new groups that we would need.

We decided to do more than just create additional selected for verification groups. Because of the reason above concerning the difficulty of changing batch posted requirements, we decided to move the logic for assigning Tax Data or W2 requirements into the groups so that we could automatically assign the appropriate requirements and they could change if the a subsequent ISIR transaction changed. This meant that for dependent students, if the verification group required tax information, there are 4 combinations of tax filing statuses:

-   Student filed tax return and Parents filed tax return
-   Student filed tax return and Parents did not file tax return
-   Student did not file return and Parents filed tax return
-   Neither Student nor Parents filed tax return

Going with this method would increase the number of tracking groups, but we felt it was worth it to enable student’s to quickly get the correct list of required documents.

Since we were creating all of these new groups, we also decided change all of the groups to have more consistent group names and to appropriately handle some types of students that were being handled manually in the past. We also use tracking groups to track students that we won’t be processing, such as students who are not yet admitted, who cancelled their admissions application, or who have with drawn. We took the opportunity to create group names for these groups that allow us to easily match then in Population Selection with a LIKE clause instead of an IN clause.

We ended up with a completely new setup of Tracking Groups that numbered significantly higher than in past years. Here is the list of tracking groups that we have defined for the year.

-   XNAPST - No Admiss App/Stu Record
-   XNACTV - Not Active
-   XNADMT - Not Admitted
-   XGRADS - Graduated
-   XWTHDN - Withdrawn
-   XCNCLD - Cancelled
-   UNODEG - UG - Non-Degree Seeking
-   GNODEG - GR - Non-Degree Seeking
-   UDLENR - UG - Dual Enrolled
-   UVISIT - UG - Visiting Student
-   UNOFIL - UG - No FAFSA on File
-   GNOFIL - GR - No FAFSA on File
-   URJISR - UG - Rejected ISIR
-   GRJISR - GR - Rejected ISIR
-   UDNOVR - UG - Dep - No Verify
-   UINOVR - UG - Ind - No Verify
-   GINOVR - GR - Ind - No Verify
-   UDV1TT - UG - Dep - V1 - Stu: T Par: T
-   UDV1TN - UG - Dep - V1 - Stu: T Par: N
-   UDV1NT - UG - Dep - V1 - Stu: N Par: T
-   UDV1NN - UG - Dep - V1 - Stu: N Par: N
-   UIV1T - UG - Ind - V1 - Stu: T
-   UIV1N - UG - Ind - V1 - Stu: N
-   GIV1T - GR - Ind - V1 - Stu: T
-   GIV1N - GR - Ind - V1 - Stu: N
-   UDV2 - UG - Dep - V2
-   UIV2 - UG - Ind - V2
-   GIV2 - GR - Ind - V2
-   UDV3 - UG - Dep - V3
-   UIV3 - UG - Ind - V3
-   GIV3 - GR - Ind - V3
-   UDV4 - UG - Dep - V4
-   UIV4 - UG - Ind - V4
-   GIV4 - GR - Ind - V4
-   UDV5TT - UG - Dep - V5 - Stu: T Par: T
-   UDV5TN - UG - Dep - V5 - Stu: T Par: N
-   UDV5NT - UG - Dep - V5 - Stu: N Par: T
-   UDV5NN - UG - Dep - V5 - Stu: N Par: N
-   UIV5T - UG - Ind - V5 - Stu: T
-   UIV5N - UG - Ind - V5 - Stu: N
-   GIV5T - GR - Ind - V5 - Stu: T
-   GIV5N - GR - Ind - V5 - Stu: N

Groups whose code begins with ‘X’ are students that will not be processed. All undergraduate groups begin with ‘U’, while graduate groups begin with ‘G’. The ‘T’ and ‘N’ indicates whether the student or parents filed a tax return for the year.

## Tracking Requirements

After determine the group structure, we began looking at the tracking requirement setup needed for  each of the verification groups.

### Verification Worksheets vs. Individual Requirement Forms

After some discussion, we decided to use individual documents for each requirement instead of creating a verification worksheet for each verification group. Two reasons directed this decision.

1.  Students will know exactly what they need to complete. In past year, students would skip a section of the verification worksheet and we would need to mark the entire requirement as incomplete. Now we can mark just the specific requirement as incomplete. The individual forms will be much shorter and easier to understand.
2.  Form maintenance will be easier. We do a lot of quality assurance checks, so a number of these requirements already exists as specific documents. Needing to maintain the individual requirements and five different verification worksheets would be more work.

One other side effect influenced our decision. We batch post a number of requirements as quality assurance measure. If we went with verification worksheet, we may request the same information twice, once on the worksheet and once on the individual requirement. By using just individual requirements, we avoid this possibility.

This was not a unanimous decision. There were strong opinions on both sides of the debate. We will be closely watching how this decision affects our processing and how students react.

### Tax Data Requirements

We will continue to use batch posting to satisfy the Tax Data requirements in the V1 and V5 verification groups when the student or parents uses the IRS Data Retrieval Tool.

### Internal Requirements

In past years, during our quality assurance checks, we would batch post a number of tracking requirements not visible to the student to indicate that we needed to review something in the ISIR record or in their NSLDS record. These requirements were assigned based on ISIR comment codes. They were vague and required the Financial Aid Administrator to search for the reason that requirement was added. We had about six of these requirements. The number of comment code triggering one of these requirements has grown significantly in the last few years.

This year, we decided to try changing this as well. We are going to using Tracking Requirements only for student facing requirements, while using holds for our internal checks. We went through our Batch Posting Rules and created about 30 hold codes to cover the various situations where we previously posted internal tracking requirements. We are hoping to reduce confusion between items that the student needs to submit and items that we need to review internally. This is a fairly substantial change in processing from previous years. It will require careful new training. We also intend to support this change with a Web Snapshot view that brings together all the data needed to easily process students, include Requirements from the various tabs of RRAAREQ, Holds, ISIR comments, and more.

This is another change that we will be watching closely to determine the success.

### High School Completion and Education Purpose Statement

In the past, we have had a fairly rigid setup for tracking requirements posted as a result of the incoming ISIR. All of them prevented packaging and disbursement. Only two requirements prevented packaging only: our internal Verify and Review requirements. We used this setup to create our reports that a student was ready to review or verify. When the student and a Disbursement Requirements Complete Date and a null Package Requirements Complete Date, they would appear on the report. This method prevented us from posting any disbursement only requirements before packaging was complete. Generally, it has worked well and we only had one situation where we ran into a problem with this method and it was easily solved.

Now with the High School Completion requirement, we have a requirement that needs to be posted along with verification, but that can’t be reasonably satisfied until after we would normally be packaging students. We decided that this would need to be a disbursement only requirement. We also decided that the Education Purpose Statement and Photo ID would also be disbursement only requirements. Because of this, we need to rewrite our reports. We now have the flexibility to assign disbursement only requirements early in the year without affecting our review or verification of students. This change also influenced the decision to change the internal requirements. In keeping with our change to holds for internal requirements, our internal Review and Verify requirements will be replaced with hold codes.

## Requirements Conclusion

Putting all these changes together, we will have a substantially different looking process for tracking requirements and verification in 2013-2014. Here is a couple of examples of the requirements posted for a particular group:

### V1 – Undergraduate – Dependent Student Selected for Verification Student and Parent filed Taxes

#### Requirements

-   Student Tax Data
-   Parents Tax Data
-   Family Size Verification Form (covers both family size and number in college)
-   SNAP Benefits Verification Form
-   Child Support Paid Verification Form
-   Additional Financial Information Form

#### Holds

-   Final Verification

### V5 – Undergraduate – Dependent Student Selected for Verification Student did not File Taxes and Parent Filed Taxes

#### Requirements

-   Student W2
-   Parents Tax Data
-   Family Size Verification Form (covers both family size and number in college)
-   SNAP Benefits Verification Form
-   Child Support Paid Verification Form
-   Additional Financial Information Form
-   High School Completion
-   Government Issued Photo ID
-   Education Purpose Statement

#### Holds

-   Final Verification

## Conclusion

This year likely involves the most significant changes to our processing setup since going live with Banner in January 2006. Our entire tracking group and requirement setup has changed and we are planning to implement Algorithmic and Period Budgeting as well. This will be a complete change to our Budget setup. I have made significant changes to a specific section in previous years such as SAP, New Student Packaging and Current Student Packaging, but never to more than one section at a time. I am curious to see how the changes are handled both by the Financial Aid staff and by the students and parents.

With this many changes, it seems likely that we will run into some issues and there will be some hiccups along the way. It will be an interesting aid year.

Once I have completed writing all of the group rules, I will post about the SQL needed to create all these tracking groups. Also look for a post about our changes to budgeting.