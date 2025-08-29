---
layout: single
title:  "Algorithmic Trading"
categories: Projects
toc: true
toc_sticky: true
toc_label: Contents
author_profile: false
---



# Algorithmic Trading Bot


```
Contents
1. Introduction
2. Ticker Selelction
3. Ticker Database
4. Discord
5. Performance
6. Conclusion
```

---

### *1. Introduction*
Algorithmic trading bots are automated systems that execute trades based on predefined strategies and algorithms. These bots analyze market data, identify trading opportunities, and place orders without human intervention with the pre-setup. By leveraging speed, accuracy, and data-driven decision-making, algorithmic trading bots can help traders optimize their strategies and manage risk more effectively.

In this blog, I'll explore the basics of algorithmic trading, discuss popular strategies, and provide insights into building your own trading bot using Python. Whether you're a beginner or an experienced trader, understanding how algorithmic bots work can give you a competitive edge in today's fast-paced financial markets.


### *2. Ticker Selection*
> Choosing the right tickers matters as much as the strategy itself.

I start by screening for the highest dollar trading value of the day `Trading Value = Volume x price`. From there, I run each candidate through a pre-configured pipeline: fundamentals, risk/return metrics, quantitative factors, DCF intrinsic value, and backtests. Based on this evidence, I decide whether to enable the bot to automatically buy and sell that ticker. 

![image]({{site.url}}/assets/images/algo_dashboard/ticker_email1.png)![image]({{site.url}}/assets/images/algo_dashboard/ticker_email2.png)![image]({{site.url}}/assets/images/algo_dashboard/ticker_email3.png)![image]({{site.url}}/assets/images/algo_dashboard/ticker_email4.png)


### *3. Ticker Database*
When I select a ticker that I think it will go down(I prefer `buy the dip`), then I add it to the dataset with fields such as ticker symbol, target buy percentage, notes/description, moving averages, and a suggested holding period derived from backtests. The database will add earnings date, PL, Win rate, moving percentage in a day and RVOL per se.

![image]({{site.url}}/assets/images/algo_dashboard/database_tickers.png)


### *4. Send commands through discord*
One feature I love is that I can send a command through discord to my database, so I can update parameters without touching the database UI, I can simply send commands like Percentage of buy, Buy Limit, Sell Limit, etc, and the system applies the changes immediately.

![image]({{site.url}}/assets/images/algo_dashboard/discord_commands.png)

### *5. Performance (3 months)*
Here's a three-month snapshot, early results were strong (peaking over 15% of return). After the market pullback I gave back some gains, but the portfolio still remains profit as it shows. The KPI dashboard are built with Apache Superset. 

![image]({{site.url}}/assets/images/algo_dashboard/Algorithmic_trading_dashboard.png/)


### *6. Conclusion*
Algorithmic trading doesn't mean "good" or "bad", but you can definitely try this thing and see how it works. If you can't monitor markets all day (I have a full time job), a well-tested, rule-based bot can help execute consistenly on your plan without straing at the screen. As always start small, validate with backtests and paper trading, and iterate thoughtfully.



















































