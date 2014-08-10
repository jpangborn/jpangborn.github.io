---
layout: post
title: "Introduction to Web Snapshot"
modified:
categories: "Banner Financial Aid"
excerpt: Learn how to use Web Snapshot to create customize views of Banner data.
tags: [banner, "financial aid", "web snapsnot"]
image:
  feature: berea-college-patio.jpg
date: 2013-03-20T00:00:00-04:00
---

Have you ever wanted to create your own form in Banner? Do you wish you could view award data along side tracking requirements? Creating customized forms is difficult, time-consuming, and requires specialized knowledge. Web Snapshot is the easier alternative. Web Snapshot allows you to create customized views with just SQL. Think of it as a method to mashup just the data you want to see. We’ll explore what it takes to get a snapshot up and running, as well as look at several of the snapshots that I have created.

## Getting Started

Web Snapshot requires some configuration before beginning. Your Banner Self-Service administrator will need to grant access to Web Snapshot. There are two levels of access that can be granted to users. One level is granted to users who will be viewing the snapshots. The other is granted to users who will be creating snapshots panes. One downside here is that anyone granted access to use Web Snapshot will have access to run all panes created. You won’t be able to restrict access to certain panes. This may limit your ability to display certain data, especially if you want different levels of access for users. This hasn’t been a big issue for us so far, but it does make us think a bit more about what panes we create.

### Snapshot Pane Creators

Users who will be creating snapshot panes need to granted the Snapshot Administrator privilege. Pane creators will need working knowledge of SQL as all panes are SQL statements.

### Snapshot Users

Snapshot Users are able to view and create snapshot layouts. Layouts combine previously created panes into a view of data. Even though users can create their own layouts, it is probably better for the Pane Creators to give some direction for layout creation.

## Snapshot Panes

Snapshot panes are the building blocks of Web Snapshot. A snapshot pane consists of one or more SQL Select Statements. Use the SQL statement to query Banner for related data for the student. Any valid SQL statement will work. Two dynamic parameters are available for use in your snapshot panes: Aid Year Code and Student’s PIDM. I will follow up this post with another going into detail about how to create panes. Below are some examples of panes that I have created:

-   Financial Aid Processing Overview (shown below.)
-   Financial Aid Award by Term
-   Financial Aid Requirements
-   Financial Aid Holds
-   Financial Aid Disbursement Edits
-   Financial Aid Budget
-   Financial Aid Comments
-   Student Account Detail by Term
-   Student Holds
-   Student Course Registration

<figure>
  <a href="/images/01-ws-pane-finaid-overview.jpg">
    <img src="/images/01-ws-pane-finaid-overview.jpg" alt="Web Snapshot Pane: Financial Aid Overview">
  </a>
  <figcaption>An example Web Snapshot Pane showing a overview of a student’s financial aid processing status</figcaption>
</figure>

## Snapshot Layouts

Once you have created some snapshot panes, they can be organized into layouts. These layouts can group panes logically for efficient display of information. I have created several layouts that are used for different purposes. Below are a couple of the layouts that I have created:

### Current Student Packaging

The Current Student Packaging layout compares data from the previous aid year with the current aid year. This is used when reviewing current student packages for the upcoming year. Users can see last year’s award and budget information along side the new year’s information.

<figure>
  <a href="/images/02-ws-pane-layout-cur-stu-packaging.jpg">
    <img src="/images/02-ws-pane-layout-cur-stu-packaging.jpg" alt="Web Snapshot Layout: Current Student Packaging">
  </a>
  <figcaption>The Current Student Packaging Layout shows data from the previous aid year and the new aid year. It is used when reviewing current student packages.</figcaption>
</figure>

### Financial Aid / Accounts Receivable View

This layout was created for our frontline staff to have an overview of a student’s information at their fingertips when answering the phones. It show all vital Financial Aid data, including an overview, requirements, budget, award, holds, and comments along with AR and Student data such as student account details by term, student holds, course registration.

<figure>
  <a href="/images/03-ws-pane-layout-fa-ar-overview.jpg">
    <img src="/images/03-ws-pane-layout-fa-ar-overview.jpg" alt="Web Snapshot Layout: Financial Aid / Accounts Receivable Overview">
  </a>
  <figcaption>The Financial Aid / Accounts Receivable layout is used by frontline staff to give them a full overview of a student that they are working with.</figcaption>
</figure>

## Upcoming Layouts

I have a list of additional layouts to create. With the changes to tracking groups and tracking requirements that I detailed in a previous post, I plan on creating a snapshot layout with a better view of the student’s record prior to packaging. I will be including tracking requirements, holds, ISIR comments, ISIR rejects and other data. This snapshot may also be used for verification.

I would also be interested in other’s ideas for snapshot layouts. Please leave a comment with any ideas for using Web Snapshot.