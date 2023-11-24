---
weight: 2
title: "Coffee and Cake, a match made in heaven: Intro to Association Rules"
date: 2023-06-09
draft: false
author: "M Wentink"
authorLink: "https://MarkWentink.github.io/DSED"
summary: "Association rules mining, aka Market Basket Analysis, helps us detect patterns in shopping behaviours, website browsing, and even medicine interaction. To explore this technique, let's set up our own coffeeshop and see whether coffee and cake is in fact the perfect match."
images: []
resources:
- name: featured-image
  src: pastries.jpg
- name: featured-image-preview
  src: pastries.jpg
#categories: ["Recommender", "Algorithms"]

lightgallery: true
---



> Not a fan of pastries? Jump straight to the section about the [Frequent itemsets and the Apriori algorith](#frequent-itemsets), or [Association Rules](#association-rules) proper.

A long-standing fantasy of mine is to open a coffeeshop one day. Somehow, I imagine this to be a quiet, peaceful existence, completely disregarding the fact that I would have to wake up at 5am to open up shop at 6:30 to get ready for the morning commuters.

Another minor point I tend to ignore when dreaming about this future is that I've never run a business or made a flat white in my life...

But let's not sweat the small stuff for now! Welcome to DSED Coffee! "The perfect brew from the Data Science crew."


### Pastries for the people

One evening, as I'm working through the pile of customer receipts from that day, I notice that a lot of people buy coffee (no surprises there), but most of those people *only* buy coffee. Wouldn't it be great if we could get them to also buy a pastry while they're here? Everything is better with a pastry! 

Just as I'm pondering a "Buy a coffee, get a pastry half price" deal, a few follow-up questions arise:

- **What percentage of coffee buyers are already buying pastries?** My deal would reduce my income from this group because I would be making their order cheaper without actually getting them to buy more. 
- **Are there other common pairings with pastries?** Maybe *tea* drinkers are more receptive to half-price pastries than coffee drinkers.
- **Are pastries actually worth focusing on?** Maybe sandwiches have more upsell potential...

You can see the problem quickly becomes quite complex. And that's just for 4 products we're considering. Imagine a supermarket or online shop with thousands of products!

### Rules-based Learning

These questions can be answered with a type of Machine Learning called **Rules-based Learning**. In the case where we explore how different events occur together, or products are bought together, we speak of **Association Rules**. 

The aim of the game is to quantify various stats about *'if A, then B'* rules such as:

>**If** a customer buys coffee, **then** they also buy a pastry.

(An important caveat that I will make repeatedly in this article is that the **rules don't imply causality**, only co-occurence. We're not saying that people buy pastries *because* they have bought a coffee, simply that they arrive at checkout wanting both those things. [Other types of rules-based learning](https://kalpanileo1996.medium.com/prefix-span-algorithm-with-pyspark-483ab3494373) do focus on *sequences* of events.)

Before we launch our promo campaign, it would be very valuable to know a few things about the *if coffee, then pastry* rule. In particular, it would be useful to know:
- **How common is this rule?** Am I wasting my energy exploring buying patterns that are too uncommon to make a difference to my bottomline?
- **How certain is this rule?** i.e. Out of the people who do buy coffee, what proportion also buys pastries? If this proportion is already high, than my half-price pastry campaign is going to do more harm than good. 
- **How strong is the relationship?** Does having one item in your basket make a customer more likely, or less likely to also have the other in their basket, or are the two items unrelated?

As we explore Association Rules more, we will refer to these questions as the *Support* (how common), the *Confidence* (how certain), and the *Lift* (how correlated) of the rule and expand on how to calculate and interpret them. 

These metrics allow us to recognise product pairings as:
- **Complementary**: The products are more likely to be bought together than we would expect from random chance (coffee and cake, tortilla chips and salsa, popcorn and soda)
- **or Substitute**: These products are less likely to be bought together. You buy one or the other, but not usually both (Coca Cola and Pepsi, butter and margarine, whole beans and ground coffee)



### Less is more

Hunting down rules with high correlations (lift) or high certainty (confidence) can greatly inform promotional campaigns, or how we organise our shop layout. We might for example want to make sure that someone walking towards the coffee aisle happens to walk past the cake section. 

The problem with exploring these patterns is that the number of possible rules very quickly becomes enormous. 


> Consider a shop that sells just 10 products.
>
>The number of **If a customer buys A, Then they also buy B** type rules we could consider is 90. (10 possible products for A, and then 9 remaining for B).
>
>But we can also consider **If a customer buys A and B, Then they also buy C** type rules, for which we would have 10\*9\*8 = 720 possibilities.
>
>Considering even bigger groups, ultimately, there are more than $10!$ ('10 factorial', which is 10\*9\*8\*7....*1, roughly 3.6 million) possible rules, but the majority of those would be extremely uncommon and not at all worth our time. 

A shop with 10 products already has millions of possible rules! A shop with 80 different products would have **more possible rules than there are atoms in the universe,** and the average supermarket sells more than 10,000 different products! We couldn't possibly look at every rule individually. (The fancy term for this behaviour is [Combinatorial Explosion](https://en.wikipedia.org/wiki/Combinatorial_explosion) and it occurs in many different fields.)

-------

We need an efficient way to come up with only the interesting rules. Specifically, we want to ignore combinations of items (itemsets) that are bought very rarely. I'm not going to bother setting up a promotional campaign if it will only appeal to one in a million customers. 

What we need is a way of finding **frequent itemsets**.


### Frequent Itemsets

The first stage in Association Rules mining is to find frequent itemsets. Let's start with what we mean by itemset.

> An **itemset** is a group of items (of any size, including a single item) that is present in a transaction (traditionally, a shopping basket).

Consider this purchase in our coffeeshop:


{{< image src="./itemsets.png" height="250px" width="580px">}}

This order (or 'basket') contains the following itemsets: `{coffee}`, `{pastry}`, `{tea}`, `{coffee, pastry}`, `{coffee, tea}`, `{pastry, tea}`, `{coffee, pastry, tea}`.

Even though coffee wasn't the only thing bought, we can still consider the `{coffee}` itemset independently. Notice that we don't care about the ordering of the basket, and we don't count multiples of the same product. Referring to sets with curly brackets is a convention from maths and programming.

>The basket contains coffee, but it also contains *the combination of coffee and pastry*. Recording how often this happens allows us to answer questions such as "Out of the coffee drinkers, what proportion also buys pastries".



So the basket above contains 7 itemsets. But are these *frequent* itemsets? To answer that question, we need receipts. A **lot** of receipts. 

-------------

Consider the following ten orders from our coffeeshop, which sells Coffee, Tea, Pastry, Sandwich, and Juice:

{{< image src="./baskets.png" height="450px" width="580px">}}

We need to record how often each item and combination of items is bought.

-------

Firstly, let's only consider itemsets of size 1, that is, individual items.

Our first step is to count in how many baskets each item features. 

Then, we divide that nr by the total nr of baskets to get the *proportion of baskets containing the item*. This is the item's **support**.

| Itemset  | nr_baskets | Support |
|----------|------------|---------|
| Coffee   | 7          | 0.7     |
| Tea      | 5          | 0.5     |
| Pastry   | 5          | 0.5     |
| Sandwich | 3          | 0.3     |
| Juice    | 1          | 0.1     |
--------

>*We* choose what we consider a *frequent* itemset, by applying some **support threshold**, say 0.2. We can then say that if coffee appears in at least 20% of our transactions, then `{coffee}` is a frequent itemset, and potentially worth our time exploring.

Our next step is to filter out uncommon itemsets. Let's stick to a support threshold of 0.2. 

We can see that Coffee, Tea, Pastry, and Sandwich all make the cut, but Juice is not common enough for us to consider: only one out of ten customers ordered juice. 

| Itemset  | nr_baskets | Support |
|----------|------------|---------|
| Coffee   | 7          | 0.7     |
| Tea      | 5          | 0.5     |
| Pastry   | 5          | 0.5     |
| Sandwich | 3          | 0.3     |
| ~~Juice~~    | ~~1~~        | ~~0.1~~     |


Itemsets of size 1 are doable, but as we saw before, as we start considering pairs and trios of items, the nr of possible combinations becomes huge quickly...

This is where the Apriori algorithm comes to the rescue. 

### Apriori

The key idea behind the Apriori algorithm is:

> If an item is rare, then any group of items including it is going to be even more rare. 
>
> Specifically, if an itemset is so rare it's not worth our while (i.e. it doesn't reach the support threshold), any bigger group involving the itemset is also not worth our while. 

Juice only occurs in 10% of our baskets, so juice *combined with something else* will be even less common. (At best it will be equally common, but it can never be more common).

This is a key insight, because it means that when we start looking for frequent itemsets of size 2, we don't have to bother looking at any pairs that involve juice.

Let's illustrate the idea with everyone's favourite diagram, the Venn diagram. 
In the below example, the left circle represents all the transactions that involved pastries, and the right circle those transactions that involved juice. 

Some of the transactions (the middle region) involve *both* pastries and juice, but because this region is the overlap between the two circles, it can never be bigger than either circle. 

{{< image src="./subsets.png" height="250px" width="580px">}}

In reality, our situation is more like the below. The juice circle is already very small, and hence the overlap region is tiny. At best every single juice transaction also involves pastries, and the juice circle would sit fully inside the pastry circle. But if `{juice}` is already too rare for us to be interested, then `{juice and pastry}` is guaranteed to also be too rare. 

{{< image src="./subsets_2.png" height="250px" width="580px">}}

------

Coming back to our example, `{coffee}`, `{tea}`, `{pastry}`, and `{sandwich}` meet the support threshold, and so we will further consider pairs of those: `{coffee, tea}`, `{coffee, pastry}`, `{coffee, sandwich}`, `{tea, pastry}`, `{tea, sandwich}`, `{pastry, sandwich}`.

Again, we count how many of our 10 transactions those pairs occur in, and then calculate the support for the item pair. 


| Itemset          | nr_baskets | Support |
|------------------|------------|---------|
| Coffee, Tea      | 2          | 0.2     |
| Coffee, Pastry   | 4          | 0.4     |
| ~~Coffee, Sandwich~~ | ~~1~~          | ~~0.1~~     |
| Tea, Pastry      | 3          | 0.3     |
| Tea, Sandwich    | 3          | 0.3     |
| ~~Pastry, Sandwich~~ | ~~1~~          | ~~0.1~~     |

We got four pairs that met the support threshold: `{coffee, tea}`, `{coffee, pastry}`, `{tea, pastry}`, and `{tea, sandwich}`.

Let's continue to sets of three items. Combining our four pairs, we now only have one valid option: `{coffee, tea, pastry}`.

How come? 

`{coffee, tea, sandwich}` is not a valid option, because we saw in the previous step that `{coffee, sandwich}` is not common enough, and so any bigger group involving those also won't make the cut. 

For the same reason, we also don't consider `{coffee, pastry, sandwich}`, nor `{tea, pastry, sandwich}` (because pastry and sandwich together is rare).

The Apriori algorithm outputs a full table of all itemsets that meet the support threshold. In this example, our final frequent itemset table is:

| Itemset  | nr_baskets | Support |
|----------|------------|---------|
| Coffee   | 7          | 0.7     |
| Tea      | 5          | 0.5     |
| Pastry   | 5          | 0.5     |
| Coffee, Pastry   | 4          | 0.4     |
| Sandwich | 3          | 0.3     |
| Tea, Pastry      | 3          | 0.3     |
| Tea, Sandwich    | 3          | 0.3     |
| Coffee, Tea      | 2          | 0.2     |
| Coffee, Tea, Pastry|2         | 0.2     |
-----

> The formal implementation of the Apriori algorithm repeats two stages: **candidate generation**, which is when we combine all the previously found itemsets to find new itemsets one size bigger, and **pruning**, where we eliminate itemsets from the list based on their support levels. 

In our example, we:

- listed all individual items, counting their occurences and calculating their support (candidate generation)
- eliminated those items that didn't meet the support threshold of 0.2 (pruning). Btw, depending on the use case, your support threshold could be much smaller, maybe less than a percent.
- created all possible pairs of the remaining (i.e frequent) items (candidate generation)
- eliminated the pairs that didn't meet the threshold (pruning)
- combined pairs into triplets, checking that all pairings within the triplet are common enough (if combining pair AB and BC, then pair AC must also be a frequent itemsets)
- continue generating and pruning, only considering a larger itemset if all the sets one size smaller within it are frequent. 

**Remember the golden rule:**

> If a pair is uncommon, then a triplet containing that pair must also be uncommon.

 ### Association Rules

 Now that we have our table of frequent itemsets, with their corresponding support scores, we can generate our "If A, then B" association rules. 

Let's focus on our initial question of "If Coffee, then Pastry". Pulling up the corresponding rows of our table:

| Itemset  | nr_baskets | Support |
|----------|------------|---------|
| Coffee   | 7          | 0.7     |
| Pastry   | 5          | 0.5     |
| Coffee, Pastry   | 4          | 0.4     |

Remember that the rule we're assessing here: "If a transaction includes Coffee, it also includes Pastry", does not imply that buying coffee *leads* to buying pastries. We're only saying that these things tend to appear in a transaction together. **We describe co-occurence, not causality**.

Let's calculate our three key stats about this rule. Remember, we were interested in:
- **Support**: how common is this rule?
- **Confidence**: how certain are we in the rule?
- **Lift**: how strong is the rule compared to if there were no special relationship between the items?

#### Support
The support of the 'If Coffee, then Pastry' rule, which we'd write `Support(Coffee -> Pastry)` is simply the support of the itemset `{coffee, pastry}`, i.e the number of transactions that contained the pair, divided by the total number of transactions. 

If you are a fan of probabilities, you would think of this as the probability of a random transaction containing coffee and pastry, i.e. $P(Coffee \cap Pastry)$.

Note this is different from the support of the individual items, `Support(Coffee)` and `Support(Pastry)`. The good news is that Apriori has precalculated all these values for us.

> Remember, `Support(A->B)` describes how common the scenario we're discussing is. Through Apriori, we have already filtered out rules that are too rare to be worth our time.

#### Confidence
`Confidence(Coffee -> Pastry)` describes: Out of the baskets that contain coffee, how many also contain pastry. 

Again, in terms of probability, you may recognise this as the probability of a basket containing Pastry, given that it contains Coffee, i.e the conditional probability $P(Pastry|Coffee)$.

This is simply the support of {coffee, pastry}, divided by the support of {coffee}, i.e. 0.4 / 0.7 = 0.57. This means that out of all the transactions that include coffee, 57% also include pastry. Or, expressed differently, if we pick a random transaction that has coffee in it, there is a 57% chance that it also contains pastry.

> `Confidence(A->B)` describes how certain we are that if a transaction involves A (which we call the *antecedent*), it will also involve B (the *consequent*).

#### Lift

Perhaps our most interest measure of the rule is Lift, which is a measure of 'how much more likely than random chance' it is to observe a group of items together. 

If we had two items, A and B, that both had a support of 0.5, we would expect to randomly see them in transactions together 25% of the time (0.5 * 0.5). If the actual Support(A -> B) is bigger than 0.25, the items seem to have a positive relationship: they appear together more often than expected.

To quantify this, we define 

$$Lift(A\to B) = \frac{Support(A\to B)}{Support(A)*Support(B)}$$

The denominator here is how often we'd expect to see the items together if there were no special relationship, hence a lift of 1 corresponds to unrelated items, a lift bigger than one shows a positive relationship (complementary items) and a lift below 1 shows a negative relationship (substitute items).

For us, $Lift(Coffee \to Pastry) = 0.4 / (0.7 * 0.5) = 1.14$

The items seem to be slightly complementary. The lift doesn't seem as strong as we may have imagined when we think about how well coffee and cake go together, but that's because both coffee and pastries are extremely common purchases, and so we would expect to see them bought together quite often anyway. 

> Lift is a multiplier on how often we see a group of items, as compared to what we would have expected if the items were unrelated.


#### Some further details

While Support and Confidence are numbers between 0 and 1, because they relate directly to probabilities, Lift can be any positive number. A lift between 0 and 1 indicates a substitute relationship between items, while lift bigger than 1 indicates complementary items. 

Support and Lift are symmetric: $Support(A \to B) = Support(B \to A)$ and $Lift(A \to B) = Lift(B \to A)$. For confidence, this is not true. If A is much more common than B, then $Confidence(B \to A)$ will be much bigger than $Confidence(A \to B)$ .

### So what does this all mean for me?

What do we now do with all this information?

We can use the insights from our market basket analysis in several ways:

- **optimisation**: If two items are commonly bought together, we want to avoid discounting them both at the same time. Discounting one of the items might encourage people to buy the pair, but discounting both is just costing us money.

- **Inventory management**: If we are running low on a specific item, we might promote an item with a substitute relationship. That way customers still feel they got what they wanted, while making our depleting stocks last longer. 

- **Upselling expensive items**: If an expensive item shows a strong relationship with a cheaper item, we can discount the cheaper item hoping that some of the extra sales will also trigger sales of the expensive item.

- **Arranging shop lay-out**: If items are commonly bought together, we might consider placing them side by side in our shop, for a smoother shopping experience. Similarly, we might place items side by side in the hope of strenghtening the lift between them. 

---------

Association Rules mining provides a great tool to better understand my customers, how they shop, what they buy, and how I can most effectively promote my items. 

I'm certainly all set for when my coffeeshop is up and running! I'd better start learning how to make coffee...


-------

Got questions? Spotted any issues in the code? Or do you want to share your own examples of implementations? Drop me a message on [LinkedIn](https://www.linkedin.com/in/mark-wentink-793217116/)