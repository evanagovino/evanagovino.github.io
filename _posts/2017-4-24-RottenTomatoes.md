---
layout: post
title: Predicting Rotten Tomatoes Scores
---

For our second project at Metis (my first solo project), we had to solve a problem involving linear regression. I decided to predict Rotten Tomatoes scores for new movies.

### Background

Rotten Tomatoes is a website that aggregates critic reviews for movies and assigns each review to be positive or negative based on the reviewer's sentiment. The site then gives a score to the movie based on the percentage of positive, or 'fresh', reviews. Movies with over 60% are 'Certified Fresh.' For instance, *The Fate of the Furious* has 142 fresh reviews out of 214 for a 66% score. It is certified fresh.

### Financial Incentive

Admittedly I originally did this because I like Rotten Tomatoes and have been using it for probably 15 years at this point to determine if a movie is worth seeing. But after thinking about it, there is definitively a financial incentive in a movie having a good score on Rotten Tomatoes.

As per Alexa, Rotten Tomatoes is the 173rd ranked state in the United States and the 507th ranked site in the world. As of 2014, there were 26 million people across the world going to the site every month. It's hard to quantify the impact of a good Rotten Tomatoes score on a movie's box office gross (that's a whole project in itself :wink:), but it has such a large reach that it should be taken seriously.

One recent example is this year's breakout hit *Get Out*, which had a budget of under $5 million and a domestic gross of $170 million. As of today, *Get Out* has a 99% score on Rotten Tomatoes, with only one negative review out of 231 (!). It seems naive to think that this had no impact on this gross.

**So**, if I wanted to invest in a film, especially one that didn't have an established brand, its potential Rotten Tomatoes score would be something I at least consider. Wouldn't it be nice to actually know that before the film was released?

### Data Sources

So, how do I begin to approach this? Well, Wikipedia has a (https://en.wikipedia.org/wiki/2016_in_film "nice page") that summarizes all movies released in a given year by release date. The page is ambiguous as far as the origin of each film / which countries it's being released it. It seems to be focused on American films, but is comprehensive enough that these questions should sort themselves out after the scrape.

I scraped data for all Wikipedia pages from 2016 back to 2000, having a substansive list of movies along with their release dates for all of these years.

Next, I needed Rotten Tomatoes info for each of these films. I set up a scraper in Beautiful Soup that attempted to pre-populate URLs for each film. The URL of a given film is pretty easy to reverse engineer - it's usually something like https://www.rottentomatoes.com/m/the_boss_baby with sometimes the year after the title of the movie. From each page I grabbed the score, along with the number of reviews counted.

I ran the list of films I got from Wikipedia through this and eliminated all films that either I couldn't get Rotten Tomatoes information for or had less than 20 reviews.

Finally, I needed background info on each film. For this I used the convenient OMDB API, which aggregates information found on IMDB. I could enter in a given movie and it would give me information on it such as release year, actors/directors involved, and much more. 

I ran my new list through the OMDB API and now had a list of movies since 2000 with background information. Overall, I had 2,572 movies in my dataset.

### Predictors

I focused on seven predictors for the model:

* Month of Release
* Genre
* Director
* Actors
* Rating (G, PG, etc...)
* Runtime
* Studio

Only one of the above predictors is numerical - Runtime. Everything else is categorical. Month of release and rating were exhaustive, meaning that every movie had one of these. Movies could have up to three genres. Directors who had directed more than ten movies since 2000 were included, along with actos who had been in more than fifteen movies and studios that had produced at least ten movies. These were set up as hotkeys, so for example Adam Sandler had his own column in the dataset and a movie with him had 1 in that column while a movie without him had 0 in that column.

Overall I had 125 predictors, 1 of them numerical.

### Model

After doing a 70/30 train/test split on the dataset, I tried four models on the test data: Linear Regression, Lasso, Elastic Net, and Ridge Regression. The latter three were set up with a five-fold cross validation on the training set.

I found that when running the model multiple times I would end up with R-Square values ranging anywhere from 22% to 30%. Much of that is due to randomness in the train/test splits along with the high number of categorial variables, which made it easier for the model to find local optima (especially in the Lasso model). 

I went with a Lasso model with the airtight logic that it had the highest R-squared and lowest MSE that I could find after a few test runs. My MSE onthe test set was 603.68 while my R-squared value was 28.25%

### Results

The results were pretty intuitive, thankfully. The intercept of the model was 17.24% and each minute of runtime had a coefficient of 0.303, meaning a movie with a runtime of 100 minutes would have a score of 47.54%. R and G rated movies did better than PG and PG-13 movies. The top performing features were famous critic bait such as documentaries, foreign films and animated films, along with studios including Disney, Fox and A24 (producers of *Moonlight*).

The biggest negative features were amusing. Gerard Butler scored the worst with a hit of -9.35%, while Relativity Studios had a hit of -8.35%. Adam Sandler, Tyler Perry and the horror genre also got negative marks.

### Next Steps

While I was happy I was able to put this together, I have a long way to go with an R-score under 30%. How can I improve this model?

One idea is to focus on what I can get via natural language processing. I can take plot descriptions for each movie and mine them for keywords (such as 'World War 2') that will indicate a higher or lower score.

Another idea is to focus on interactions between features - something I hadn't looked at in the model. For example, if a movie is rated R, that's good, if it's a comedy, that's bad, but what if it's an R-Rated comedy? How do those two features interact with each other?

A third idea is to use models that are less prone to overfitting, such as random forest. If I can implement this model, I can increase the number of features - specifically in the actor, director and studio categories.

### That's it!

I had a pleasure doing this and am looking forward to updating this model as I try out some new techniques on it!