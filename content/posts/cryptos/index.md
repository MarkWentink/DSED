---
weight: 1
title: "Two sides of the same coin? Visualising crypto-currency price correlations"
date: 2024-04-06
draft: false
author: "M Wentink"
authorLink: "https://MarkWentink.github.io/DSED"
summary: "To what extent can we diversify a crypto portfolio?"
images: []
resources:
- name: featured-image
  src: coins.jpg
- name: featured-image-preview
  src: coins.jpg
#categories: ["analysis"]

lightgallery: true
---


As of April 2024, there are around 10,000 crypto-currencies out there. Seemingly plenty of choice if you are looking to invest in something. But how can you go beyond simply clicking buy on something with a funny name? And is there in fact that much to choose from, or are all coins ultimately the same?

In this article, I will focus on the question of how different the biggest crypto coins are from a statistical perspective. I will leave the specific value propositions of different coins aside for now, and purely focus on a key statistical technique when comparing price trends, correlation analysis. 



# The illusion of diversification

I should start by saying I am not a financial analyst nor in any way qualified to give advice. This is not advice, but only my findings from a brief statistical analysis on crypto-currencies. 

Let me start by clarifying what I understand under diversification, i.e. the practice of spreading money across multiple investments. 

First and foremost, the purpose of diversification is to reduce risk and provide more stability to a portfolio overall. If one of the investments goes south, this should only account for part of the money, and hopefully other investments will make up for the loss. 

By spreading investments across different stocks, different industries, different global regions, and different assets, we reduce the impact of any one event on our bottomline. 

But we can also create the **illusion of diversification**. If I buy a stake in several different vinyards all from the same region, I may feel like I have spread out my risk, but if that region is hit by a cold snap, all those vinyards are likely to suffer, and it's still my whole portfolio that's affected.

Instead, we could invest in vinyards from different regions, so that not all of them will be affected by the same weather events. But this still leaves us exposed to changes in for example the global demand in wine. Better to also invest in other agricultures, or better yet, something completed unrelated, like a stake in some real estate, a tech company, or government bonds. 

But let's say we've got our eye on crypto, and we have set aside *some* (not all) of our capital to invest. What do we buy and to what extent can we still diversify our portfolio?


# A statistical angle

Once you make it onto a crypto exchange, there are several hundred tokens available to buy at the click of a button. 

How do we know which one to buy, and **does the choice even matter?** That second question is where the stats come in. 

Let's focus on the top 30 cryptocurrencies by market cap. After removing near-identical tokens such as staked Ether (StETH) and wrapped Bitcoin (wBTC), we are left with 25 top tokens. They are presented below sized by market cap and coloured by type of token (blue for transactions, green for smart contracts, etc...)

{{< image src="./market_caps.png" height="500px" width="1160px" caption="Top 25 crypto-currencies sized by market cap">}}


The first thing to note is that the market is still dominated by BTC, accounting for about 60% of the market. This should already raise some concerns about diversification, because whatever happens to bitcoin is bound to affect the rest of the market. 

Working off the last 3 years of daily prices of these tokens, we can start looking for strong correlations, i.e. tokens that systematically move up together, and move down together. 

Let's check correlations between top tokens. When exploring correlations across several features, a popular way to present this as a heatmap, where a red square indicates a strong relationship between a pair of tokens. 

{{< image src="./heatmap.png" height="500px" width="1160px" caption="Correlations between pairs of the top 25 tokens. Deep red is strong correlations ">}}

This is not easy to look at, at all, but a few things do stand out:

- the whole graph is quite red, indicating that, in general, all cryptos coins are quite strongly correlated to each other
- there are a few tokens that are relatively immune to correlations, typically the stablecoins (USDT, USDC, DAI)

We can do better than this. Rather than plotting correlations by colour in a grid, let's use a tree diagram:

{{< image src="./tree.png" height="500px" width="1160px" caption="Crypto-currencies correlations clusters">}}

In a tree diagram like this, all the tokens are listed at the bottom. The closer to the bottom the branches of currencies join up, the more strongly correlated they are. This allows us to detect groups of currencies that tend to behave similarly. 

For example, the left-most group, in orange, are the stablecoins. These are somewhat correlated to each other, as they are all linked to the US dollar, but they don't join up with the other currencies until the very top of the tree. They are very indepedent from the rest when it comes to price fluctuations. 

Most of the transaction-focused tokens appear in the green group: ADA, DOT, BCH, XRP, LTC. These are all very strongly correlated to each other, to the point where investing in multiple of these is an illusion of diversification, you might as well just buy one. 

Interestingly, DogeCoin, a meme coin with really no value proposition at all, is very strongly linked to the transactional tokens. As transactional tokens make up a significant portion of the market, this might indicate interest in DogeCoin follows general interest in the crypto market. 

{{< image src="./network.png" height="500px" width="1160px" caption="Crypto-currencies correlations as a network">}}



# Bring in context

-------

Got questions? Spotted any issues in the code? Or do you want to share your own examples of implementations? Drop me a message on [LinkedIn](https://www.linkedin.com/in/mark-wentink-793217116/).

-------

Photo Credit: Harrison Broadbent, Unsplash