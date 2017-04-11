---
layout: post
title: MTA Turnstile Project
---


For our first project at Metis, we acted as consultants for a nonprofit tech group called WomenTechWomenYes (WTWY). WTWY is a group promoting the interests of women in technology. The group has an annual gala at the beginning of the summer each year, and needed our help to help raise awareness for the event. 

###The Problem

Their plan was to use street teams outside of subway stations to collect e-mail addresses for people potentially interested in the gala. They were generally looking for people passionate about their cause (and perhaps passionate about giving money towards their cause), but in general wanted to build awareness and reach for their brand.

Our job? To find the best subway stations for them to do this out of.

###Initial Thoughts

For a data scientist, this is an incredibly open-ended problem. Note how general the request is - the timeframe is vague (beginning of summer), there is no idea of how many people they need to fill the gala or how many email addresses they want to collect to achieve they end (though they mentioned they generally want to build awareness and reach). We don't know anything about logistics. How big is their streetteam? How many hours can their streetteam commit? Would they rather us focus on pure quantity (thus building awareness and reach), or find people more likely to donate? Is there a minimum amount of money they need to raise at this gala?

Clearly we need to make some assumptions about what they are looking for.

###Assumptions

We will assume that WTWY is looking for a higher quantity of email addresses collected rather than a smaller, but more targeted list of email addresses from people with a higher likelihood of being donors. We will assume they'd rather err on the side of quantity than quality.

With that said, we will assume that WTWY is looking for a list of email addresses from people who are interested in their cause. We think that people working at tech companies would be more interested in this cause than people who don't work at tech companies, or are in the city as tourists or for recreational purposes.

We think that people who attend similar types of galas may be interested in this gala.


###The Plan

We will go into the MTA data from the past three months to find the most popular subway stations by total weekday traffic. Focusing on data from just the work week will let us focus on people going to and from work.

We will also find a list of the locations of the top 25 tech companies in New York.

We will also find the locations of any upcoming galas in New York.

We will filter the list of top stations by proximity to either a top tech company or an upcoming gala. If a station is within a quarter-mile radius of either of these, we will consider it eligible to work at.

From there, we will sort these stations by the most popular hourly intervals, in order to recommend the best times for the street team to go to these locations.

###Results

First, we went through the past three months of MTA data to find the 50 most popular stations by total weekday volume.

Then, we found the locations of the top 25 tech companies of New York by total employee account.

Then, we found the locations of all upcoming events on Eventbrite for the next four months (April - July) through the Eventbrite API.

Then, we found a list of 26 stations that were within a quarter-mile or either a top-tech company or an upcoming Eventbrite event.

Here are the 15 stations that were both in the 50 most popular stations and 26 stations close to events or tech companies.

<img src="/../images/stationlist.png" width="200" />

![Map of Stations](/../images/stationlist.png =200x)
