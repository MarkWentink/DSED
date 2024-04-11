---
weight: 1
title: "Data parsing, validation, and wrangling"
date: 2024-04-11
draft: True
author: "M Wentink"
authorLink: "https://MarkWentink.github.io/DSED"
summary: "Something about parsing different file types"
images: []
resources:
- name: featured-image
  src: elections.jpeg
- name: featured-image-preview
  src: elections.jpeg
#categories: ["analysis"]

lightgallery: true
---

Ward-level election results are a goldmine for political analysis, but suffer from a problem that haunts data analysts in all industries: inconsistent data recording. Standardisation in terms of how voting outcomes should be recorded is limited, usually resulting in different file types, naming conventions and ordering of information. Before we can start mining for insights, we need to read in all of these files, find the information we're looking for, standardise the format, and collect it all in one place. 
Before we jump into how to make that happen, let's briefly talk about voting. 

Not too fussed about elections? Go straight to the [file parsing and data processing](#exploring-the-files).

-----------

In Scotland, Members of the Scottish Parliament are voted for at constituency level with a [first-past-the-post](https://en.wikipedia.org/wiki/First-past-the-post_voting) system. There are 73 constituencies for Scottish parliament elections, and 59 constituencies for UK parliament elections. 

Local councillors, however, are elected in 354 'electoral wards'. This is the level at which local council elections are typically studied. With vote counts at ward level, we can study how different political parties are represented locally, and how close some of the races were. 

This sounds like it would culminate in a very tidy spreadsheet, where you simply have a table structure `| ward | candidate | vote_count |`. Things are complicated a bit by the Scottish local voting system of preferential voting.


# Single transferable voting: preference ranking

In a traditional first-past-the-post system, people cast a vote on a single candidate, and the candidate(s) with the most votes wins. A commonly raised challenge with this system is that it discourages people from voting on smaller or less popular political parties. Voting on a candidate that doesn't stand much chance of gaining an actual majority can be seen as throwing away your vote, even though that may well be your preferred candidate.

Instead, Scotland handles a [Single Transferable Voting](https://en.wikipedia.org/wiki/Single_transferable_vote) system. Instead of casting their ballot for one candidate, voters rank a list of candidates by preference. Your vote is initially counted towards your first preference. However, if your first choice does not get enough votes, and gets eliminated, your vote is transfered to your next favourite candidate. This means that voting for a candidate you feel strongly about but may seem unlikely to get enough votes is not a wasted vote. Even if the candidate doesn't make it, your vote still counts towards your second best option.

This does make for significantly more complex election result analysis. Instead of simply having a list of candidates with corresponding vote counts, we have a large number of possible preference rankings, each offering potential insights not just in which way voters are leaning, but also into what their second or third choice would have been, i.e. where a small nudges such as more intensive campaigning are most likely to influence election outcomes. 


# Exploring the files

Once voting results have been made public, we can gather the outcomes of each ward and end up with a neat folder with 354 files, which somehow have to be turned into one comprehensive dataset. 

Let's start with a quick look inside a file. Below are the first and last few rows of a .txt file:

{{< image src="./example_file.png" height="300px" width="1160px" caption="Top and bottom rows of an election record file" >}}

At first glance, the voting results look like random strings of numbers. A bit of puzzling (and asking someone who knows) later, we've found out that every record starts with the number of people voting that exact preference order, followed by numbers representing each of the candidates, which are listed at the bottom.

For example, the second row, `17 1 3 6 5 0` signifies that 17 people voted that exact combination, which is candidate 1 (Fairlie) as first preference, followed by candidate 3 (Lindsay), 6 (Quinn), and 5 (Quin). The 0 represents an end-of-line token.

Depending on what kind of analysis we (i.e. the stakeholders) are after, there are many ways to aggregate and simplify these records. Maybe we'd choose to simplify voting patterns to only a top 3 so that we can aggregate some of the records.

In this case, the ask was to retain the full preference order, but to replace the numbers not with the candidate names, but with their party affiliations. 

The file shown was rather tidy. Some of the data formatting and validity issues we came across:
- records are stored in 5 different filetypes: `csv`, `xls`, `txt`, `blt`, `xlsx`
- .csv files were not comma separated, so were recognised as containing only one column
- Files had rogue whitespace is various places
- Candidate names may or may not be in quotation marks
- Independent candidates had their party affiliations left empty
- .txt files included escape characters

# Wrap up



-------

Got questions? Spotted any issues in the code? Or do you want to share your own examples of implementations? Drop me a message on [LinkedIn](https://www.linkedin.com/in/mark-wentink-793217116/).

-------

Photo Credit: Harrison Broadbent, Unsplash