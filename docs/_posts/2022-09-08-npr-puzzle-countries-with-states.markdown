---
layout: post
title:  "NPR Puzzle: Name Those Countries"
date:   2022-09-08 11:01:00 -0800
categories: NPR-Puzzle lapply reduce
---

## Name Those Countries

Every Sunday, [NPR Puzzlemaster Will Shortz](https://www.npr.org/series/4473090/sunday-puzzle), provides a listener challenge, a unique puzzle to solve at home on [NPR's Weekend Edition Sunday](https://www.npr.org/programs/weekend-edition-sunday/). Correct submissions are entered into a drawing and the lucky winner gets to play along with the on air challenge the following week!!! 

Here's my second [NPR Sunday Puzzle](https://www.npr.org/series/4473090/sunday-puzzle) attempt! 

The September 4, 2022 puzzle sounded like a good challenge to use and practice R. 

Here's the puzzle, courtesy of [Blaine's Puzzle Blog](https://puzzles.blainesville.com/2022/09/npr-sunday-puzzle-sep-4-2022-postcards.html): 

>Name two countries with a total of 12 letters that, when spelled one after the other, form six consecutive state postal abbreviations. What are the two countries?

## Using R 
My R code uses two really nice Tidyverse packages, readr and stringr. These packages are some of my most used packages. Stringr, in particular, is a great package for working with strings. I also use a CSV file found online from [Kaggle](https://www.kaggle.com/datasets/fernandol/countries-of-the-world). This code is a one-time use so it's not as clean as should be and has several errors. I decidied to keep the code sloppy as it better illustrates my thought process. There's a lot of practice done here with lapply, reduce, and anonymous functions. Finally, this also uses a built in R dataset, state.abb, a list of US state abbreviations. 

## Step 1 -- Combine Countries together 

My first thought? How hard can this be? I need a vector of all countries and then I need to combine each country together with each other country and limit the end result to 12 characters. There can't be that many countries combined that are 12 letters in length! 

Load up the readr and stringr packages and read in the vector Country from my [country.csv](https://github.com/dephilli/NPR-Puzzle-Using-R/blob/master/countries%20of%20the%20world.csv) file.

{% highlight r %}
library(readr)
library(stringr)

#bring in countries
country <- read_csv("countries of the world.csv", col_select = "Country")
{% endhighlight %}

Now, to make my life easier, I'm going to convert everything in country to uppercase using the stringr package 

{% highlight r %}
country$Country <- stringr::str_to_upper(country$Country)
{% endhighlight %}

My next step is a bit complicated. Partially because I have combined several different things together. This is going to return a vector of all the countries mashed together with all other countries. 

{% highlight r %}
country_full <- unlist(lapply(1:length(country$Country), function(x) {
  paste0(country$Country[x], country$Country)
}))
{% endhighlight %}

Boy, that's a lot of stuff going on. I'm going to break it down into three subparts, The anonymous function, the lapply, and the unlist. 

First, the anonymous function

{% highlight r %}
function(x) {
  paste0(country$Country[x], country$Country)
}
{% endhighlight %}

This function is taking a single vector item and then using paste0 (which pastes objects together without a separator) to paste that one country to all other countries. 

If we made this a named function like so:

{% highlight r %}
country_glue<-function(x) {
  paste0(country$Country[x], country$Country)
}
test_it_out<-country_glue(1)

{% endhighlight %}

When I add lapply and the argument (1:length(country$Country)) this returns a list for each country. The big flaw here is that the country pastes to itself. I decided for a puzzle to not worry about this and ignore any cases in my final lookup. However, for production code this should be fixed and I'll hopefully add a fix to this at a later date!!! 

Finally, I added an unlist argument to the beginning which takes our 200+ list of vectors and returns it to a single vector. Wow! 51,000+ records. 

## Step 2 -- Get rid of long countries. 

Now, I know the answer can't possibly include many countries. Clearly, the answer doesn't include the United States of America (way more than 12 characters). Now, I could have eliminated all countries longer than about 9 characters in length. However, I didn't think it mattered to get to the final answer so I decided to take these large strings out at the end. 

{% highlight r %}
country_combined <- country_full[nchar(country_full, "char") == 12]

{% endhighlight %}

Okay, this should give a list we can look at and answer our question. 

Whoops! This list is still huge! Over 3,000 entries. This is still too many to look at.

So, I have a lot of options here. I could remove all countries that are repeat countries "CANADACANADA", etc. However, I think I should move on and use this list to meet part 2 of the question. The entire string needs to be made up of consecutive U.S. state abbreviations. 

## Step 3 -- Find Consecutive State Abbreviations 

R has state abbreviations as a built in dataset. This makes life pretty great. I don't need to download a list.

{% highlight r %}
state.abb
{% endhighlight %}

Certainly, if I use stringr over the first two letters, I will have a list I can eyeball and see my answer! 

{% highlight r %}
country_combined[substr(country_combined,1,2) %in% state.abb]
{% endhighlight %}

This code uses substr and the first two characters from country_combined and sees if the substring matches any of the state abbreviations. So, ALBANIAARUBA pops up. AL=Alabama. However, this isn't the right answer, the next needed substring is BA and (BA %in% state.abb) is FALSE. 

What I need to do instead is check each group of letters and see if they are in the state.abb dataset. 

So, I made another lapply with an anonymous function.

{% highlight r %}
countries_with_state_abbr <- lapply(seq(1, 11, by = 2), function(x) {
  country_in_state <-
    country_combined[substr(country_combined, x, x + 1) %in% state.abb]
  country_in_state <- data.frame(country_in_state)
})
{% endhighlight %}

The anonymous function:

{% highlight r %}
function(x) {
  country_in_state <-
    country_combined[substr(country_combined, x, x + 1) %in% state.abb]
  country_in_state <- data.frame(country_in_state)
  }
  {% endhighlight %}

The anonymous function is looking at a two character substring starting at x and ending at x+1. So, if we pass the number one in as x we will get the first two characters of country_combined. We'll grab all the trues and return a single column dataframe (more on this in Step 4).

The anonymous function is plugged into a lapply statement with a strange argument: seq(1,11, by=2). This argument takes all the odd numbers between 1 and 11 plugs each into the anonymous function and returns all matches. The first pass looks at the first two characters of country_combined and sees if they are in the state.abb data. The second pass looks at the third and fourth letters and sees if they match the state.abb data, and so forth. The final return is a list of six dataframes. Each dataframe contains the countries that match part of the substring to the state abbreviations. 

## Step 4 -- Turning the list of dataframes to the answer 

Now, how can I figure out which countries are on all six lists. The easist way is through the Reduce function:

{% highlight r %}
Reduce(function(d1, d2)
  merge(d1, d2),
  countries_with_state_abbr)
  {% endhighlight %}

This final step merges each dataframe and returns matched records. This step returns two results. One is bad MALAWIMALAWI. It's 12 letters, and each two letter pair is a state abbreviation (Massachusetts, Louisiana, Wisconsin )However, this is clearly not correct! I can't repeat the same country twice. 

The second response is DENMARKSPAIN, it's 12 letters, two different countries, and the states are Delaware, New Mexico, Arkansas, Kansas, Pennsylvania and Indiana. Boom! Final Answer! With any luck, next stop Weekend Edition Sunday! 

There are likely many different ways to solve this issue, reach out with any questions or criticism.

[Full code available here](https://github.com/dephilli/NPR-Puzzle-Using-R/blob/master/countries%20with%20state%20name.Rmd)
