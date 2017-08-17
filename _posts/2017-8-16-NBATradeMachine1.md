---  
layout: post  
title: Custom NBA Trade Machine Part One 
---

While I have some extra time on my hands, I wanted to do some fun stuff with this upcoming NBA season. I've flirted with making prediction models for the NBA before; a few years ago I tried creating optimal lineups for FanDuel with Excel's 'Solver' function and uh...let's just say I can't retire from that one. But as a huge NBA fan and someone trying to break into data science, it seems silly to not do some fun stuff with basketball analytics.

The ESPN Trade Machine has been a nerd staple for what has to be ten or fifteen years at this point - essentially it gives you a dashboard to let you come up with fake trades and keeps you within the bounds of the CBA rules so that only trades that can actually be done will be allowed to pass. It's fun and easy to use but still limited - I don't think I've seen it updated at all since it was created, either functionality or design-wise. For instance you can't throw in picks to the trade (nor get an approximate value for what a pick is worth) and the only metrics they give you are PER for each player (even worse, during the offseason the PER is 'N/A'). The Trade Machine then gives you 'Hollinger's Analysis' that project the change in wins for each team, but the process behidn that metric is unknown. Shoutout to John Hollinger, but it sounds like it could use some updating (also John Hollinger hasn't worked at ESPN for several years which tells you how old it is).

So I want to try my hand at creating a trade machine that I'll host at this site. There are a few steps to do this:

1) Pull all of the salary data for each player from Hoops Hype or another website with salary information and create a database that will store this information as well as periodically update it.

2) Write the functionality to propose trades and the logic to accept or reject them based on CBA rules.

3) See what analytics I can put in - perhaps I can create forecasts for each player for the 2017 season based on past data. I could also create forecasts for future draft picks based on past data.

4) Develop a Flask app as a front end so that people can use it in a web broswer.

Step 3, in my opinion, will be the most challenging. I'll have to think a lot about creating an accurate forecasting model for NBA players.

But I like this project - it combines building a database, building a Flask app, and doing prediction modeling.

I'll update as I go along. Looking forward to this one! 

