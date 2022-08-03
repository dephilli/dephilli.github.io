---
layout: post
title:  "Trimming Whitespace in a Dataframe"
date:   2022-08-01 08:44:00 -0800
categories: data clean up
---

## Too Much Whitespace

In my work, I am constantly pulling data from a roboust and ancient RDBMS. Oftentimes, the data items I want to use are filled with excess whitespace that is either not valuable for data analysis, or makes data analysis downright painful. Now, there are many ways to get rid of trailing whitespace. Perhaps the best approach is to get rid of it at the source. Most SQL languages have a trim function where individual data types can be trimmed. However, it can be a little daunting to add a trim function to each data type. 

This, however, is a bit of a pain. Luckily, R comes to recuse with a handy function called trimws. 

{% highlight r %}
df$lots_of_whitespace <- trimws(df$lots_of_whitespace)
{% endhighlight %}

This works really well and trims off all the extra whitespace. But, what if I have multiple columns? 

Lapply to the rescue! 

Let's say I have three columns I wish to update at once. I can create a variable with those three columns and then run a lapply to fix each column. 

{% highlight r%}
cols_to_trim <- c("col1", "col2", "col3")
df[cols_to_trim]<-lapply(df[cols_to_trim], trimws)
{% endhighlight %}

This will take each preselected column, and trim all the whitespace. Great approach! However, I still need to preselect my columns. What if I want to trim all the character columns? For that, I built a simple function that should work with any dataframe.

{% highlight r%}
trim_all_ws <- function(dframe) {
  cols_to_trim <- names(loc_hist[sapply(loc_hist, is.character)])
  dframe[cols_to_trim] <- lapply(dframe[cols_to_trim], trimws)
  return(dframe)}
{% endhighlight %}

This simple function takes the dataframe, and creates a new temporary dataframe called names, which includes only the character columns. Then it creates a value vector of the names of the character columns, runs the same lapply as above and removes all whitespace from all character columns. 

