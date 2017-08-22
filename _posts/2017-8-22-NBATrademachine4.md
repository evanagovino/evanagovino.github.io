---  
layout: post  
title: Custom NBA Trade Machine Part Four
---

Today I was able to expand the capability of the command-line program from two teams to four teams.

This took more time as I had anticipated, as I had hard-coded a lot of the code (<a href='http://www.evan.ag/NBATradeMachine3/' target="_blank">see here</a>) to only work for two teams, so I had to rewrite a number of functions to be able to take in a variant amount of teams, as well as add in the ability to point a player to different destinations.

Now the same functions for two teams work for four. You can propose a trade and then add/remove players from each team to make the trade work. I had some fun proposing a three-way Knicks/Cavs/Nuggets trade that would get Melo in Cleveland and Kyrie in Denver.

<img src="/../images/nbacommandline_screenshot7.png" width="800" />

There are still some kinks I need to work out. You can trade one player multiple times, and it doesn't allow you yet to change the destination for a given player once it's set (unless you remove that player and then add him back in).

I also need to work on adding more rules to the trades as discussed previously.

Overall this was a good lesson in making something scalable from the start, as I probably spent too much time working on something that should have been relatively easy. I still need to make a few adjustments to make it scale higher than four teams, but I'd guess that 90% of the work is done on that end.
