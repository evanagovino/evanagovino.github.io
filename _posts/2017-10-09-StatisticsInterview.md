---
layout: post
title: Interview Questions - What is the Central Limit Theorem?
---

## What is the Central Limit Theorem?

The central limit theorem is the idea that if you have a population with an arbitrary distribution, the mean of repeated samples from this population will be equal to the mean of the population itself.

Let's explore this by creating an arbitrary discrete distribution. We will pick a number from the following list:

[1, 1, 1, 2, 3, 4, 4, 5, 6, 6]

Think of it as rolling a weighted die that has more of a chance of giving you a 1, 4, and 6 than a 2, 3 and 5.

The mean of this list is 3.3 while the standard deviation of this list is 1.9.

Now let's say I randomly take a sample of ten with replacement from this list. Maybe the ten I take will look like this.

[4, 1, 1, 5, 3, 4, 4, 4, 4, 1]

The mean of this sample is 3.1, which is decently close to the actual mean of the list.

If I randomly take another sample, say it looks like this:

[4, 4, 2, 4, 4, 6, 1, 5, 2, 6]

<img src="/../images/discrete_distribution.png" width="800" />

This mean of this sample is 3.8, while the mean of my two means is now 3.45.

If I take ten samples like this, I find that the mean of my ten samples will be 3.31, which is incredibly close to the mean of the actual distribution.

If I take 1,000 samples like this, I find that the mean of the 1,000 samples is 3.31, with a distribution looking very close to a normal distribution.

<img src="/../images/normal_distribution.png" width="800" />

What's even crazier is that the standard deviation of the normal distribution is 0.61. While this seems arbitrary, I can square it to find that my variance is 0.3742. Squaring the standard deviation from the original distribution gives me a variance of 3.61. If I divide 3.61 by 0.3742, I find an answer of 9.65. If I round that, I find that it's equal to the sample size of ten that I decided to take.

That means that the variance of my distribution of means is equal to the variance of my original distribution divided by the sample size!

Below is some code you can use to run this on your own that uses this distribution. Just put in the sample size and the number of samples you want to take, and it will output the mean and variance of your sample distribution.

```
import random
import numpy as np

def make_new_list(my_list, sample_size):
    blank_list = []
    i = 0
    while i < sample_size:
        x = random.choice(my_list)
        blank_list.append(x)
        i += 1
    return(np.mean(blank_list))

def make_master_list(my_list, sample_number, sample_size):
    blank_list = []
    i = 0
    while i < sample_number:
        x = make_new_list(my_list, sample_size)
        blank_list.append(x)
        i += 1
    return blank_list


def create_results(distribution, sample_size=10, sample_number=1000):
    print('Mean of Distribution:', np.mean(distribution))
    print('Standard Deviation of Distribution:', np.std(distribution))
    list_of_means = make_master_list(distribution, sample_number, sample_size)
    print('Mean of Means:', round(np.mean(list_of_means),2))
    print('Standard Deviation of Means:', round(np.std(list_of_means),2))
    print('Your Sample Size Should Be Close To:', round(np.power(np.std(distribution)/np.std(list_of_means),2)))

l = [1, 1, 1, 2, 3, 4, 4, 5, 6, 6]
create_results(l)
```


