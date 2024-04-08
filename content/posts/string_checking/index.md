---
weight: 2
title: "Rucksack Re-org: Advent of Code Throwback 1, String Parsing"
date: 2023-11-24
draft: false
author: "M Wentink"
authorLink: "https://MarkWentink.github.io/DSED"
summary: "In this Advent of Code challenges, we introduce some simple string parsing and checking with sets, and prepare some ready-made function that we may be able to re-use in later challenges. "
images: []
resources:
- name: featured-image
  src: andrew-neel-backpack.jpg
- name: featured-image-preview
  src: andrew-neel-backpack.jpg
#categories: ["Advent of Code", "Strings"]

lightgallery: true
---

# The most wonderful time of the year

One week from now, on December 1st, is the moment we have all been waiting for! No, it is not the first day Mariah Carey is back on the radio, that was straight after Halloween. It's [Advent of Code](https://adventofcode.com/) time! A series of daily coding challenges in December that can be solved in any language. I will code in Python throughout these posts. 

In anticipation of all this excitement, I'm revisiting some of last year's problems, to give you a flavour of what the challenges involve, and hopefully to get you coding! 

First up: **AoC 2022 day 3**


# Rucksack reorganisation

Advent of Code challenges tend to have somewhat convoluted storylines, so let's first dig into the text to see what we are actually being asked to do. Rather than copying the lengthy description, I will link to the website and paraphrase:

[AoC 2022 Day 3](https://adventofcode.com/2022/day/3) is a string parsing problem. We are told that our problem input is a list of text strings representing rucksack contents. Each letter (case sensitive) in the string represents some item. We are further told that rucksacks have two compartments of equal size, so the string can be split in half to represent the contents of each compartment. Crucially, items of the same type (i.e. the same character) are supposed to go in the same compartment, but each rucksack has one mistake. We need to identify which item in each rucksack does not follow the rule (i.e. which item features in both halves of the string) and score the bad items following the rule: a-z becomes 1-26, A-Z becomes 27-52.

Let's look at an example and then summarise the rules:

>One example entry is as follows:
>
>`vJrwpWtwJgWrhcsFMMfFFhFp`
>
>The two compartments (halves) of this rucksack are `vJrwpWtwJgWr` and `hcsFMMfFFhFp`. The 'bad' item, i.e. the only one that appears in both compartments, is `p`, which corresponds to number `16`. Hence this rucksack gets a score of `16`, which will be summed to the others for the final answer.

#### Rule summary

- Each rucksack is given as a string of characters representing items. We can have multiple of the same item in a rucksack. 
- Each rucksack contains two compartments, the two halves of the string.
- Each rucksack will have one item type (one character) that appears in both compartments
- We need to identify this character, convert it into a score `a-z --> 1-26` and `A-Z --> 27-52`.
- The answer is the sum of the scores across all rucksacks. 


# Let's get coding

Establishing the rules of the game is half the work. We know we're going to need to be able to read in strings, check them for specific characters, and convert characters into scores. 

Each of these tasks should lend themselves nicely to a function. 

#### The plan

It often helps to write out the overall program first, abstracting away the details as function names for you to worry about later. I usually do this in plain english code comments first (also known as pseudo-code), and then will in the corresponding python code. 

For us, our program will look something like this:

```python
def day_3(input_file_path):

    # load in the puzzle input as a list from the specified file path
    data = load_data(input_file_path)
    # prep a list to keep track of scores
    scores = []

    # loop through the puzzle input
    for rucksack in data:

      # identify the bad item
      bad_item = find_bad_item(rucksack)

      # calculate its score
      score = score_item(bad_item)

      # save the score
      scores.append(score)


    # sum all scores to answer the challenge
    answer = sum(scores)

    return answer
```

That wasn't so bad! Now, small detail: we don't have any of those functions yet! Let's get cracking on those `load_data()`, `find_bad_item()` and `score_item()` functions. 



#### Data loading

Let's start with a function for loading in the input data, which is given as a .txt file. It's a good idea to standardise this function as much as possible, as we will be using it pretty much every day during AoC.


```python
def load_data(file_path):
    """
    Parses a .txt file from 'path', and appends each line as an entry to a list. 

    Input: .txt file 
    Output: list of lines
    """
    with open(file_path) as f:
        data = [line.strip() for line in f.readlines()]
    
    return data
```
The function accesses a specified filepath and reads the file with the built-in `open()` function. `.readlines()` is an iterable object, giving us the lines of the file one by one. In this problem, the lines don't really need any preprocessing, so I'm just stripping off any possible whitespace with `.strip()`. The whole thing is wrapped in a list comprehension to return a list of cleaned strings. 

Remember the list comprehension is equivalent (but runs faster) to a for loop:

```python
for line in f.readlines():
  data.append(line.strip())
```

Out comes `data`, which looks something like this:

```
['WwcsbsWwspmFTGVV',
 'RHtMDHdSMnDBGMSDvnvDjtmpTpjTFggpmjmTFggTjmpP',
 'vtCSGRMBDzHddvBHBzRhrlcZhlLzWNlqblhzcr',
 'shhszHNHHZWqSzVNdClMjlFjBBbNTB',
    ......]
```

Not particularly nice to look at, but easy enough for a program to iterate through and do some work on. 

Next up, we need a function that can identify the 'bad' item in each rucksack.

#### Finding bad items

Now that we have our input, we can work through it in a for loop. For each rucksack, we need to find which character appears in both halves. 

There is a quick trick here if you know about [sets](https://www.w3schools.com/python/python_sets.asp). A set is a list of unique values, and sets come with two particularly useful functions: `union` and `intersection`. `intersection()` returns only those items that appear in both sets, which is exactly what we want here!

All we need to do is split our rucksack into two halves, create a set of unique values from each half, and use intersection() to identify the character that both sets have in common. 


```python
def find_bad_item(rucksack):

  # Figure out where to split the halves
  midpoint = len(rucksack)//2

  # Separate the halves
  half_1 = set(rucksack[:midpoint])
  half_2 = set(rucksack[midpoint:])

  # Look for characters that appear in both halves. This returns a set.
  shared_items = half_1.intersection(half_2)

  # There should be only one. We can extract it from the set if we turn it back into a list.
  bad_item = list(shared_items)[0]


  return bad_item
```
To make our code more robust against weird scenarios, it could be good to put in some safeguards.

Fortunately, in this problem, we are advised that rucksacks are always of even length, so we don't have to worry about having halves of unequal length.

However, accidents happen: what if there is a typo in the puzzle input, and I end up with a rucksack that has no overlapping, or multiple overlapping items? 

We can account for this with something like this:

```python
.....
if len(shared_items) == 1:
  bad_item = shared_items[0]
else:
  print("Error: unexpected number of shared items")
.....
```

equivalently, we can use an `assert` statement to throw an actual error:

```python
.....
assert len(shared_items) == 1, "Unexpected number of shared items"
.....
```
Upon running, a bad rucksack would trigger the following Python error:

`AssertionError: Unexpected number of shared items`.

It's always good to think about what could go wrong with the usuer input or with unexpected data, and try to account for that as much as possible.

Lastly, we need a function that can convert the identified items into scores.

#### Scoring



For this, we can use the `ord()` function, which converts a character into its [ASCII code](https://www.asciitable.com/). All uppercase, and lowercase letters, have consecutive ASCII codes. 

We just need to account for the fact that we start from code `65` for `A`, which is supposed to be `27` so we need to subtract `38` from all uppercase scores. 

The lowercase alphabet is even further down the list, and starts from `97` for `a`, so we will subtract `96` from these.

```python

def score_item(item):
    """
    Takes in a single character, and returns its score leveraging its ASCII value. 

    Input: item: a single character as string
    Output: value: its score as an int

    """
    if ord(item) in range(65,91):
        value = ord(item)-38
    else:
        value = ord(item)-96

    return value

```

This now let's us calculate the score for a single backpack. We'll just need to keep track of these scores in a list so we can sum them together later. Alternatively, we can add them to sum running tally. 


#### Putting it all together

Now, all that's left is to put everything together into one python file or notebook, run `day_3(file_path)`, and out comes the magical number. 

You submit your answer onto the webpage, and are rewarded with a beautiful gold star! ... and part 2 of the challenge....

Another day, another coding challenge. 

-------

Got questions? Spotted any issues in the code? Or do you want to share your own examples of implementations? Drop me a message on [LinkedIn](https://www.linkedin.com/in/mark-wentink-793217116/).

-------

Photo Credit: Andrew Neel, Unsplash