---
layout: post
title: "Formatted Missing Information Letters in Word"
modified:
categories: "Banner Financial Aid"
excerpt: Learn how to use Banner and Microsoft Word to generate well formatted Missing Information Letters.
tags: [Banner, "Financial Aid", Letters, Requirements, Word]
image:
  feature: autumn-leaf.jpg
date: 2012-02-11T11:25:13-04:00
---

While working with Banner, I have often come across schools that would like to create nice looking Missing Requirements Letters in Microsoft Word. This is difficult proposition when extracting data from Banner. All that you get in the data file is text. I’ll go over the process that I have adapted to resolve this. It involves some forethought in designing your variables and a Word Macro.

## Step 1: Designing your Requirements Variable

First we need to setup the variables that will extract the requirements information from Banner for each student. By choosing some unique characters to place in the variable at key points, we can offset portions of the variable for later use. Below is an example variable:

<!-- {% gist jpangborn/8000689 %} -->

{% highlight sql %}
SELECT '_' || nvl(rtvtreq_long_desc, 'None') || ':_ ' || nvl(rtvtreq_instructions, '')
FROM rtvtreq, rrrareq
WHERE rrrareq_aidy_code = &aidy_code
  AND rrrareq_sat_ind = 'N'
  AND rrrareq_treq_code = rtvtreq_code
  AND rtvtreq_info_access_ind = 'Y'
ORDER BY rtvtreq_long_desc
GROUP BY rtvtreq_long_desc
{% endhighlight %}

At a high level, this variable retreives the Long Description and Instructions for all requirements that are not satisfied and should be visible to the student. It also sorts the results by Long Description and groups them so that you only get one result if the student has the requirement twice.

More important is what is going on in the Select line. We are surrounding the Long Description with underscores. This will make it easy later in Word to find and apply formatting to just the Long Descriptions.

## Step 2: Setup Mail Merge Document

Next, we need to create our Missing Requirements document. This the template that will be your main mail merge document. The key place in this document is the location of the requirements. Below you find an Example Missing Requirements Letter. Notice that I decided to make the requirements a bulleted list. In this template, there is only one bullet. All of the requirements will be inserted into this bullet, and then after the mail merge, we will use a macro to handle reformatting the requirements so that the Description is bold and there is one requirement per bullet. In addition to the requirements, you would also need to add other variables for student name and any other data elements that you want to appear in the final letter.

## Step 3: The Macro

Finally, we need to setup the macro that used the reformat the requirements:

<!-- {% gist jpangborn/8000713 %} -->

{% highlight vbnet %}
Sub ReformatTrackingMessages()

Selection.Find.ClearFormatting
With Selection.Find
.Text = “_*_”
.Replacement.Text = “”
.Wrap = wdFindContinue
.Format = False
.MatchCase = False
.MatchWholeWord = False
.MatchAllWordForms = False
.MatchSoundsLike = False
.MatchWildcards = True
End With

Do While Selection.Find.Execute
Selection.Font.Bold = True
Loop

Selection.HomeKey Unit:=wdStory, Extend:=wdMove

Selection.Find.ClearFormatting
With Selection.Find
.Text = “_”
.Replacement.Text = “”
.Wrap = wdFindContinue
.Format = False
.MatchCase = False
.MatchWholeWord = False
.MatchAllWordForms = False
.MatchSoundsLike = False
.MatchWildcards = False
End With

Selection.Find.Execute Replace:=wdReplaceAll

Selection.Find.ClearFormatting
With Selection.Find
.Text = “^l”
.Replacement.Text = “^p”
.Wrap = wdFindContinue
.Format = False
.MatchCase = False
.MatchWholeWord = False
.MatchAllWordForms = False
.MatchSoundsLike = False
.MatchWildcards = False
End With

Selection.Find.Execute Replace:=wdReplaceAll

End Sub
{% endhighlight %}

The first With Selection statement finds all the long descriptions surrounded by underscores, and change the font to bold. Then we find all of underscores and remove them. Finally, we use a side effect of the Banner Variable extraction process. Banner puts a line break character in between each requirement when it extracts the variable. We replace all of the line breaks with new paragraph characters. This has the effect of triggering a new bullet for each requirement. Take this macro and save it as a subroutine in the mail merge document that you created and you will be able to run it.The above Word Document should already contain this macro.

## Step 4: Put it all together

Now we can put it all together to get a nice looking Missing Requirements Letter. Run the Banner Letter Generation Process to generate the data file for you letter. Open you Mail Merge document and completing the merge, selecting Edit Individual Letters. Once the new document with your letters appears, you can run the ReformatTrackingMessages() macro to take care of your formating. Now you are set to go. If your feeling really ambitious, you can use the Mail Merge Events feature in Word to have the macro auto run after the mail merge completes.