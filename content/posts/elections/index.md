---
weight: 1
title: "Every Vote Counts: Data parsing, validation, and wrangling of local election outcomes"
date: 2024-04-11
draft: False
author: "M Wentink"
authorLink: "https://MarkWentink.github.io/DSED"
summary: "Inconsistent data recording is a problem that you are guaranteed to come across as a data scientist. Here, we look at an example of writing object-oriented code for repeatable and interpretable data processing."
images: []
resources:
- name: featured-image
  src: elections.jpeg
- name: featured-image-preview
  src: elections.jpeg
#categories: ["analysis", "ETL"]

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

Once voting results have been made public, we can gather the outcomes of each ward and end up with a neat folder with 319 files (some wards were excluded from the analysis), which somehow have to be turned into one comprehensive dataset. 

Let's start with a quick look inside a file. Below are the first and last few rows of a .txt file:

{{< image src="./example_file.png" height="300px" width="1160px" caption="Top and bottom rows of an election record file" >}}

At first glance, the voting results look like random strings of numbers. A bit of puzzling (and asking someone who knows) later, we've found out that every record starts with the number of people voting that exact preference order, followed by numbers representing each of the candidates, which are listed at the bottom.

For example, the second row, `17 1 3 6 5 0` signifies that 17 people voted that exact combination, which is candidate 1 (Fairlie) as first preference, followed by candidate 3 (Lindsay), 6 (Quinn), and 5 (Quin). The 0 represents an end-of-line token.

Depending on what kind of analysis we (i.e. the stakeholders) are after, there are many ways to aggregate and simplify these records. Maybe we'd choose to simplify voting patterns to only a top 3 so that we can aggregate some of the records.

In this case, the ask was to retain the full preference order, but to replace the numbers not with the candidate names, but with their party affiliations. 

The file shown was rather tidy. Generally, these are some of the data quality issues we came across:
- records are stored in 5 different filetypes: `csv`, `xls`, `txt`, `blt`, `xlsx`
- .csv files were not comma separated, so were recognised as containing only one column
- Files had rogue whitespace is various places
- Candidate names may or may not be in quotation marks
- Independent candidates had their party affiliations left empty
- .txt files included escape characters

# Code Structure

I opted for an object-oriented approach for readability and re-usability. The diagram below illustrates the overall process.

&nbsp;  


{{< image src="./ETL_process.png" height="350px" width="1160px" caption="process diagram for ward data cleaning" >}}

&nbsp; 
The code was split into:

- A **Ward_data superclass**, with functionality for:
  - reading in the data file (this method needs to vary per filetype)
  - checking the file loaded correctly
  - locate and extract the ward name
  - extract the list of candidates
  - split up the preference order combinations
  - replace the candidate id in the preference order by their party name
  - export as a Pandas DataFrame to concatenate to the dataset
- **Three sub-classes**, only differing in their read_data method, to handle different file types
- A **create_object function**, that checks the current file type and creates the appropriate WardData instance
- a **main function** which
  - checks the content of a folder
  - loops through each file 
  - creates and calls the corresponding class
  - keeps track of progress, potential duplication, and isolates files that caused issues into a separate folder for further inspection.

This structure means that a lot of the complexity of data cleaning and validation can be split into smaller tasks in the Ward_data methods, while the `main` function stays very readable:

```python
def main():
    
    # set up
    master_data = pd.DataFrame()
    processed_wards = []
    nr_files = len(os.listdir('../data/'))
    process_count = 0
    duplicate_count = 0
    raw_files = glob.glob('../data/*')

    
    for file_path in raw_files:
        
        # create the appropriate class instance for the filetype, 
        # read in the data, and identify the ward    
        WD = create_object(file_path)
        WD.read_data()
        WD.extract_ward_ID()

        # If the ward has already been processed (i.e. its a duplicate file),
        # move into a separate folder and skip
        if WD.ward_ID in processed_wards:
            print('Ward already processed')
            shutil.move(file_path, '../duplicated/'+file_path.split('/')[-1])
            duplicate_count += 1
            continue
        else:
            processed_wards.append(WD.ward_ID)

        # replace candidate ids with party affiliation, create a sorted dataframe
        WD.extract_candidates()
        WD.split_votes()
        WD.replace_parties()
        WD.combine()
        WD.sort_by_count()
        
        # Append ward to overall dataset
        master_data = pd.concat([master_data, WD.clean_data])
      
        # progress tracker
        process_count += 1
        if process_count % 10 == 0:
            print(f'Processed {process_count} / {nr_files} files')
    
    
    print(f'{process_count} files processed. {duplicate_count} duplicates isolated.')
    
    return master_data
```

It took some tinkering to ensure enough checks were in place to ensure each file could be parsed correctly, but ultimately this resulted in a clean dataset of a quarter million rows: 350 odd wards with around 800 combinations of voting preference each, sorted by corresponding counts. 

-------

The resulting dataset is easily taken forward into aggregations and filtering to answer questions like:
- Which parties won seats outright and in which wards?
- Which parties won seats only because first preferences dropped out?
- Which wards had the tightest races, and within those, where could a small shift in preferences have swung the election outcomes?

More from a behavioural angle, we can ask questions like:
- Which political parties are most compatible in people's minds (i.e. most interchangable in preference order)
- Comparing to national elections, we can ask how differently people vote in a preferential voting system vs a first-past-the-post system.

# Wrap up

Data cleaning and validation always requires a level of tinkering and trial and error. By compartimentalising code, we can improve its clarity and re-usability, and more easily troubleshoot the process next time data recording changes again.

-------

Got questions? Spotted any issues in the code? Or do you want to share your own examples of implementations? Drop me a message on [LinkedIn](https://www.linkedin.com/in/mark-wentink-793217116/).

-------

Photo Credit: Harrison Broadbent, Unsplash