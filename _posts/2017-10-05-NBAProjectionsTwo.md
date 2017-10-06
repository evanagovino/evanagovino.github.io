---  
layout: post
title: NBA Projections Part Two: Rookies
---

The first order of business in projecting NBA performance for the upcoming season is predicting the performance of rookie players.

As noted in my <a href="http://www.evan.ag/NBATradeMachineProjection/" target="_blank">previous post</a>, the metric that I'll be predicting for players in Win Shares.

I'll parse that into two different metrics. The first is Win Shares/48, which finds the number of win shares a player will contribute in 48 minutes. The second is Minutes Played, which is the number of minutes a player plays in a season. By combining these two metrics, I can find a player's total Win Shares for the season, but it's important to separate them here, as the factors that contribute to a player's efficiency can be much different than the factors that contribute to their minutes played, especially for a rookie who may not play as many minutes as other players.

*** Pulling Data ***

First I'll need to pull data for this from Basketball Reference.

The first set of data I'll be pulling is individual player performance. In order to do this, I'll scrape the rosters of every team from the 2017 season going back to the 2007 season to get ten years of data. I'll then go through every player on every team and scrape their individual page to get their year-by-year stats. I'll be sure to put something in the code that stops it from scraping the same player's page twice (since I'm pulling for multiple years, the same players will come up multiple times, but pulling for multiple years allows me to get stats from players who retired or otherwise left the league.)

The second set of data I'll be pulling is draft position. Since I'm assuming that there will be a relationship between draft position and my two variables, I'll need to get the draft position of every player for the last several drafts. I'll also pull this for the previous ten years.

The third piece of data I'll be using is the previous pulled year-by-year record data from the <a href="http://www.evan.ag/NBATradeMachineProjection/" target="_blank">previous post</a>. Since I have a hunch that a player's win shares may be affected by the success of their team (i.e. a similarly effective player will have more win shares on the Golden State Warriors than on the Brooklyn Nets), this may prove useful. I'll be using this data from the previous ten years.

*** Predicting the WS/48 ***

First I'll look at WS/48 data for all rookies for the past ten years.

<img src="/../images/rookies_ws48_distribution.png" width="800" />

The results look something like a normal distribution with a mean of 0.005 or so.

<img src="/../images/position_ws48.png" width="800" />

What was most surprising to me was that there was no clear relationship between the position a player was drafted in and his WS/48. Rookies who are drafted later don't perform any worse on average than rookies who are drafted earlier.

<img src="/../images/record_ws48.png" width="800" />

To me it was also surprising that there was no clear relationship between the team's record from the previous record and a player's WS/48.

<img src="/../images/playerposition_ws48.png" width="800" />

There does, however, appear to be a relationship between different positions and WS/48 for rookies, with rookie C/PFs having a higher WS/48 than the other three positions.

<img src="/../images/SCREENSHOT_1207.png" width="800" />

Here is the linear equation we can use to predict the WS/48 scores for the rookie class.

*** Predicting the Minutes Played ***

<img src="/../images/minutes_played_distribution.png" width="800" />

The distribution of minutes played for rookies has a clear right skew, although this may not matter for predictive purposes unless the residuals are also skewed.

<img src="/../images/position_vs_mp.png" width="800" />

There's a clear negative slope here, which is at least intuitive. The lower a player is picked, the less minutes they will play in their rookie season. It's worth noting that while a linear relationship works here, it may be possible to find a more accurate relationship by transforming the predictor variable.

<img src="/../images/position_minutesplayed.png" width="800" />

There's no clear difference here in the quartiles of the different positions, although it's worth noting that the whiskers of the PG and SG positions go higher than the other positions.

<img src="/../images/SCREENSHOT_1241.png" width="800" />

Here both the position a player is drafted and the player being a PG are significant variables.

<img src="/../images/SCREENSHOT_1243.png" width="800" />

However squaring the 'position' variable results in an increase of the R-squared variable from 20.7% to 26.1%.

Now we can use these formulas to predict the WS/48 and MP values for each pick in the draft. The one thing we need to address is undrafted players.

Since there's no draft position for undrafted players, one idea is to use Age as a predictor.

<img src="/../images/age_WP48.png" width="800" />

There doesn't seem to be much of a relationship here. How about position?

<img src="/../images/Pos_ws48_undrafted.png" width="800" />

Not much differentiating the different positions here either.

<img src="/../images/SCREENSHOT_1048.png" width="800" />

This is a low R-squared value, but both the PG and SG variables are significant here. For now this is the best we can do for predicting WS/48 scores for undrafted rookies.

<img src="/../images/age_MP_undrafted.png" width="800" />

Here we can see that there is a positive relationship between Age and Minutes Played for undrafted rookies.

<img src="/../images/mp_position_undrafted.png" width="800" />

Again, there's not much differentiation between minutes played for different positions.

<img src="/../images/SCREENSHOT_1055.png" width="800" />

Here, the best model only has Age as a significant variable.

Now that we have prediction models for WS/48 and MP for rookies, we can make predictions on the rookie class of 2017.

<img src="/../images/SCREENSHOT_1029.png" width="800" />

Above is the rookie class of 2017. The model likes forwards and centers more than guards, but gives more minutes played the higher someone is drafted. Still, the model has players like Lauri Markkanen and Bam Adebayo projecting to make a higher contribution than Markelle Fultz and Lonzo Ball.

<img src="/../images/SCREENSHOT_1030.png" width="800" />

There's not much of a contribution from undrafted players.

<img src="/../images/SCREENSHOT_1031.png" width="800" />

And here are players from previous years. The model has Ben Simmons having far and away the biggest contribution out of all the rookies, which I happen to agree with.

That's all for now - next I'll look at veterans and predicting both their minutes played and win shares.


