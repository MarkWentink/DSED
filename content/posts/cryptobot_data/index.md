---
weight: 1
title: "Build-a-Bot part 1 : A framework for crypto simulations"
date: 2024-05-01
draft: False
author: "M Wentink"
authorLink: "https://MarkWentink.github.io/DSED"
summary: "As a young and potentially not fully efficient market, crypto-currencies are a fertile land for exploring trading algorithms. To test these, we start by building a framework that can generate and simulate trading strategies. The framework is build with a focus on automation and scalability."
images: []
resources:
- name: featured-image
  src: crypto.jpg
- name: featured-image-preview
  src: crypto.jpg
#categories: ["OOP", "Simulation"]

lightgallery: true
---

Crypto-currencies are a very young and crucially, accessible, market with ultimately very little oversight on what gets traded.
It's sensitivity to hype and the bandwagon effect seemingly make it a highly inefficient market. Whether or not you believe that leaves room to make quick money, it certainly makes crypto markets fertile land for simulating and testing trading strategies and models.   

With that in mind, I have built a framework for creating and comparing crypto trading bots. If you would like to have a play around with the MVP, you can find [Build-a-Bot](https://build-a-bot.streamlit.app/) on Streamlit. (Note you'll probably want to be on a computer for the app, the mobile version is, as of yet, a mess). 

# Why an app?

So why go through the trouble of building an app rather than just running a bunch of simulations and sharing the outcomes?

First of all, selfishly, for practice. When it's so easy to build a stack of notebooks that end up so chaotic and long that you might as well start from scratch, I wanted to create something that could scale; that I could build on and add functionality to without adding more chaos. 

Secondly, because I enjoy tinkering, and I want to give others a chance to tinker as well. The main focus of Build-a-Bot is, unsurprisingly, the [Bot Creator](https://build-a-bot.streamlit.app/Bot_Creator) where you can define your own trading strategy and portfolio allocation (within the limited realm of the functionality currently implemented). The created bots can then be compared in terms of performance and risk. As I add more functionality and strategy options, hopefully this will turn into a playground where people can experiment to find which trading strategies maximise return and minimise risk.

If you want to have a go yourself, go ahead to the [app](https://build-a-bot.streamlit.app/). Otherwise, you can read more about the code implementation below. 

# Code Structure

The framework is made up a series of interacting classes, broadly split up into two areas: managing and updating the dataset, and creating and simulating portfolios.

&nbsp;

{{< image src="./code_diagram.png" height="450px" width="1080px" caption="Overview diagram of the code implementation of Build-a-Bot. Greyed out functionality is yet to be implemented.">}}

The Data Management side relies on the Yahoo Finance API to retrieve pricing data for a given list of crypto-currencies.
The list was generated from [Yahoo Finance's Top 30 Cryptos list](https://finance.yahoo.com/u/yahoo-finance/watchlists/crypto-top-market-cap/) and filtered down to ignore tethered coins and stablecoins. 

A `YahooInterface` class ensures that we only make calls to the API when it is actually required, and only on relevant tokens, and processes the response into a dataframe. 

The `PriceData` class is in charge of the dataset. It saves it in memory, and updates it when required. Specifically, it will check what the latest entry in the current dataset is, and will only call the Yahoo Interface if the dataset is not up to date. 


On the simulation side, we mainly have `Strategies` and `Portfolios`. Strategy objects have access to a portfolio's current holdings, and the price data, and determine, for a specific point in time, what should be sold or bought. Currently implemented strategies are `Hold` and `Rules`.

`Hold`, as the name suggests, does nothing. Your portfolio simply holds on to the initial allocation and tracks its value over time.

`Rules` offers a series of buy and sell criteria that the user can define. For example, a `buy-consecutive` rule, would look for crypto coins that have been going up in value for a certain number of consecutive days, and will then buy some to try and ride the wave. A `sell-reversal` rule will sell a coin once its price has started dropping for some number of days. There is plenty more here that could be implemented in terms of decision-making rules, but ultimately, this strategy monitors prices until some criteria are met, at which point a trade is triggered. 

Next steps in terms of strategies would be to implement `Model-Based` strategies, where an autoregressive model, an XGBoost, or an RNN would be trained and their predictions used to trigger trades. 

Lastly, the `portfolio` object is the bot itself. It has a holdings DataFrame which contains its current and past allocation and it has functionality to execute buy and sell trades, to check its exposure, and to valuate its current state in terms of annualised return and volatility. 

------

Want to read some actual code? Visit the [GitHub repo](https://github.com/MarkWentink/crypto_simulator).


# So what's next?

While I'm happy with the MVP, that is what it is. There is lots more I would like to add, and there are a few pain points that need sorting. 

In terms of major functionality I'd want in a next version, main focus areas are:
- Expanding the Rules-based strategy
- Adding an initial Model-based strategy
- Create functionality to cross-validate strategies over different time windows
- Add in-depth evaluation of each bot, with a detailed log of the trades it made, how many of those were profitable, ...
- Expanding the Market Overview page with more in-depth correlation analysis and individual coin drill-down

Do you have ideas as to functionality you would like to see? Let me know!


-------

Got questions? Spotted any issues in the code? Or do you want to share your own examples of implementations? Drop me a message on [LinkedIn](https://www.linkedin.com/in/mark-wentink-793217116/).

-------

Photo Credit: Ant Rozetsky, Unsplash