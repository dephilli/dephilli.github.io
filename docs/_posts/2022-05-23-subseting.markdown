---
layout: post
title:  "R Subsetting Vectors and Dataframes"
date:   2022-05-23 08:44:00 -0800
categories: base r
---

## Simple Guide to Subsetting

R allows for a lot of different ways to do what we need to do. There's the data.tables way, the Tidyverse way, the Base R way and any other number of unique and different approaches. Coming from SAS, I was most comfortable with the Tidyverse way of doing things. However, I've found that the underlying Base R approach is sometimes a bit easier. 

## Subsetting 

I find the easiest way to subset is using simple logic and the square brackets []. At first, the syntax is a bit tricky but after a little bit of practice, I can build new sets of data with ease. 

## Subsetting a Vector 

I have a vector with NA records and I want to get rid of the NA records. This is rather simple, I subset the records and negate (!) my selection argument. 


{% highlight r %}
vector_clean <- vector_dirty[!is.na(vector_dirty)]
{% endhighlight %}

Simple! 

## Subsetting a Dataframe 

This is where the syntax can get a bit trickier and ugly. However, with some practice, this gets pretty simple. Since we have two dimensions (rows and columns) we have to separate our subset with a column [do row work, do column work]. In many cases, I want all the columns and a subset of rows, this is where I get confused a lot. 

Here's the basic syntax pulling dirty data from the dataframe dirty_df removing all NA values and returning all columns. 

{% highlight r %}
clean_df <- dirty_df[!is.na(dirty_df$dirty_vector),]
{% endhighlight %}

## Example with Real Data

Okay, keeping this very simple, I'm going to use the Lahman baseball dataset. Install the package Lahman and read the teams file as a dataframe:

{% highlight r %}
install.packages("Lahman")
teams <- Lahman::Teams
{% endhighlight %}

This is a huge dataset going back to the 1870s. I'm only interested in really good teams since 1960. First, I am going to filter for teams from 1960 and later. 


{% highlight r %}
teams<-teams[teams$yearID>=1960,]
{% endhighlight %}

Now, I want to only look at teams that won 95 games. 

{% highlight r %}
teams<-teams[teams$W>=95,]
{% endhighlight %}

This is great but I have way too much information, I'm only interested in the team, the year, their wins and losses, and if they were they won the World Series. For this, I can use the column selectors. The Lahman data is great with names for each column so picking these out and changing their order is simple. 

{% highlight r %}
teams<-teams[,c("name", "yearID", "W", "L","WSWin")]
{% endhighlight %}

The great thing is, we can do all this work in a single step with ease. 

{% highlight r %}
teams<-teams[teams$yearID>=1960 & teams$W>=95,c("name", "yearID", "W", "L","WSWin")]
{% endhighlight %}
