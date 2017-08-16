---
layout: page
title: Resources
permalink: /resources/
---

Here I wanted to collect various websites that contain resources for data science and programming. I've been asked everything under the sun in interviews, and wanted to create a reference page that is a one-stop-shop for any data science-related issues.

### Ensemble Methods

<a href="https://discuss.analyticsvidhya.com/t/what-is-the-fundamental-difference-between-randomforest-and-gradient-boosting-algorithms/2341/3" target="_blank">Random Forest vs. Gradient Boosting</a> - The two videos at the bottom of this post (via Udacity) explain how both the Random Forest and Gradient Boosting alogorithms work. This is the clearest explanation of the two I've come across.

### Case Studies

<a href="http://blog.yhat.com/posts/predicting-customer-churn-with-sklearn.html" target="_blank">Predicting Customer Churn With Sci-Kit Learn</a> - This is incredibly helpful blog post that addresses a churn problem in many steps. First the author builds a classification model that optimizes accuracy for a churn dataset. Then he moves to recall. Then he looks at churn probability rather than a binary classification and combines the probability with each customer's value so that it can calculate an expected loss. Finally he develops production level code that spits out a CSV file of results. Great post, with all of the code included.

<a href="https://www.analyticsvidhya.com/blog/2016/02/time-series-forecasting-codes-python/">A Comprehensive Beginner's Guide to Create a Time Series Forecast</a> - One topic that seems relatively undercovered to me in a lot of data science tutorials is time series models. This post, complete with Python code, introduces you to concepts like stationarity, moving averages, decomposition and ARIMA forecasting. It's a good jumping off point to learn more about doing time series forecasts.