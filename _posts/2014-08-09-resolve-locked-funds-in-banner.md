---
layout: post
title: "Resolve Locked Funds in Banner"
modified:
categories: "Banner Financial Aid"
excerpt: Did you just run disbursment, only to have a bunch of funds lock. Learn how to resolve locked funds.
tags: [Banner, "Financial Aid", Funds]
image:
  feature: messiah-college-snow.jpg
date: 2014-08-09T17:56:18-04:00
---

You’ve just run disbursements for the first time this semester, and a bunch of students have funds that didn’t memo or disburse. Quite likely, the reason the funds did not disburse is because the fund locked during the disbursement process. The most common reason for a fund to lock is because it ran out of money. Here are the steps that I use to resolve fund locks.

## Step One: Review RPEDISB Report

The first place to start is by reviewing the output of RPEDISB, the disbursement process. A section of the report includes all of the students and funds that could not be disbursed because the fund is locked. Make a note of all of the funds that are locked.

## Step Two: Review RFIBUDG

Next, take a look at RFIBUDG, the Fund Budget Form, for the first fund from the first step. We’re looking for the dollar value of the current offer for the fund. Note this dollar value for the next step.

## Step Three: Update RFRMGMT

Now, we’ll head to RFRMGMT, the Fund Management Form and make some changes to the fund setup. For the fund from the previous step, compare the current offer value noted in the last step with the Budget Allocated on RFRMGMT. If the value on RFRMGMT is less, update it to match the current offer value from RFIBUDG. Make sure not to update the Available to Offer or Over Commitment Percent. This is a common mistake. Banner will allow you award up to the Available to Offer amount, but the Disbursement process will lock a fund once the memo or paid amount for the fund reaches the Budget Allocated. Once you have updated the Budget Allocated, switch to the Disbursement Locks tab and Record Remove the lock for the term for which you are disbursing aid.

## Step Four: Repeat as Needed

Repeat steps two and three for each of the funds noted in the first step. This should resolve the locks for all of funds.

## Step Five: Rerun RPEDISB

Resolving the locks for the funds will not automatically disburse the aid. You will need to rerun RPEDISB for the term. Review the output to make sure that the funds did not lock again. Also, remember that a student may have other issues that prevent aid from memoing or disbursing, so following these steps is not a guarantee that a particular student’s aid will be disbursed.
