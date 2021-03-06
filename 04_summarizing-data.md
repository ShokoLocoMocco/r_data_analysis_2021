Summarizing data
================

[\<\<\< Previous](03_piping.md)

-----

## summarize()

![Source: dplyr cheatsheet](images/summarise.png)

Every data frame that you meet implies more information than it
displays. For example, `spotify` does not display the average energy of
a particular season, but `spotify` certainly implies what that number
is. To discover the number, you only need to do a calculation:

``` r
spotify %>% 
  filter(season == "Spring") %>% 
  summarize(avg_energy = mean(energy))
```

    ## # A tibble: 1 x 1
    ##   avg_energy
    ##        <dbl>
    ## 1      0.618

`summarize()` takes a data frame and uses it to calculate a new data
frame of summary statistics.

### Syntax

To use `summarize()`, pass it a data frame and then one or more named
arguments. Each named argument should be set to an R expression that
generates a single value. `summarize` will turn each named argument into
a column in the new data frame. The name of each argument will become
the column name, and the value returned by the argument will become the
column contents.

### Example

I used `summarize()` above to calculate the average energy of songs in
the spring, but let’s expand that code to also calculate

  - `max` - the spring song with the most energy
  - `sum` - the total energy of all songs in the spring

<!-- end list -->

``` r
spotify %>%
  filter(season == "Spring") %>%
  summarize(
    avg_energy = mean(energy),
    max_energy = max(energy),
    total_energy = sum(energy)
  )
```

    ## # A tibble: 1 x 3
    ##   avg_energy max_energy total_energy
    ##        <dbl>      <dbl>        <dbl>
    ## 1      0.618      0.972        1607.

Wow. Look at that efficient use of pipes (`%>%`) AND `summarize()`\!

**Exercise 1**

Compute these three statistics:

  - the average loudness of songs in the fall (`mean()`)
  - the maximum danceability of any song in the fall (`max()`)
  - the minimum energy of any song in the fall (`min()`)

Remember, you will have to use `filter()` to filter for fall songs\!

-----

### summarize by groups

How can we apply `summarize()` to find the average energy for each genre
in `spotify`? You’ve seen how to calculate the average energy of a
genre, which gives us the answer for a single genre of interest:

``` r
spotify %>% 
  filter(season == "Spring") %>% 
  summarize(avg_energy = mean(energy))
```

    ## # A tibble: 1 x 1
    ##   avg_energy
    ##        <dbl>
    ## 1      0.618

However, we had to isolate the season from the rest of the data to
calculate this number. You could imagine writing a program that goes
through each season one at a time and:

1.  `filter`s out the rows with just that season
2.  applies `summarize` to the rows

Eventually, the program could combine all of the results back into a
single data set. However, you don’t need to write such a program; this
is the job of `dplyr`’s `group_by()` function.

## group\_by()

![Source: dplyr cheatsheet](images/group_by_1.png)

`group_by()` takes a data frame and then the names of one or more
columns in the data frame. It returns a copy of the data frame that has
been “grouped” into sets of rows that share identical combinations of
values in the specified columns.

### Using group\_by()

![Source: dplyr cheatsheet](images/group_by_2.png)

By itself, `group_by()` doesn’t do much. It assigns grouping criteria
that is stored as metadata alongside the original data set. If your
dataset is a tibble, as above, R will tell you that the data is grouped
at the top of the tibble display. In all other aspects, the data looks
the same.

However, when you apply a dplyr function like `summarize()` to grouped
data, `dplyr` will execute the function in a groupwise manner. Instead
of computing a single summary for the entire data set, dplyr will
compute individual summaries for each group and return them as a single
data frame. The data frame will contain the summary columns as well as
the columns in the grouping criteria, which makes the result
decipherable:

``` r
spotify %>%
  group_by(covid_period) %>% 
  summarize(min_loud = min(loudness))
```

    ## # A tibble: 2 x 2
    ##   covid_period min_loud
    ## * <chr>           <dbl>
    ## 1 post_covid      -19.2
    ## 2 pre_covid       -23.0

To understand exactly what `group_by()` is doing, remove the line
`group_by(covid_period) %>%` from the code above and rerun it. How do
the results change?

**Exercise 2** Calculate the average danceability of *top 40* songs for
each season. The structure of your code should look like this:

``` r
spotify %>% 
  filter(rank <= ****) %>% 
  group_by(****) %>% 
  summarize(mean_dance = mean(****))
```

## mutate()

The `mutate()` function is another highly useful tool for extracting
unseen insights from your dataframe. While `select()` allows you to
choose columns and `group_by()` allows you to summarize rows, `mutate()`
enables you to create, modify, and delete columns. This is an extremely
flexible function, so we’ll only be able to demonstrate a small portion
of its functionality here.

A very common use of `mutate()` is to change the type of a variable. For
instance, `spotify` has a variable called `rank` that is currently
classified as a number.

``` r
class(spotify$rank)
```

    ## [1] "numeric"

Although the variable `rank` is a number, we consider it an ordered
factor. The number 1 song is ahead of the number 2 song, which is ahead
of the number 3 song, etc., so the order matters, but doing an operation
like dividing all of the values by two doesn’t make any sense. In
practice, if we were to run a regression with `rank` as a number we
would get a different answer than using `rank` as an ordered factor. So,
let’s use `mutate` to change `rank` from a number to an ordered factor.
We’ll name this new data frame `spotify_rank`.

``` r
spotify_rank <- spotify %>% 
  mutate(rank = as.ordered(rank))

class(spotify_rank$rank)
```

    ## [1] "ordered" "factor"

You can also create new rows from existing rows\! For instance, if we
wanted to create a new composite metric called `raw_power` that includes
the interaction of `energy` and `loudness`, we could do something like
this:

``` r
spotify_power <- spotify %>% 
  # since loudness values closer to zero are louder, we need to standardize the values by their minimum value
  mutate(raw_power = energy * (loudness - min(loudness)))

spotify_power %>% 
  select(energy, loudness, raw_power) %>% 
  # the head function allows us to visualize the first few rows of data
  head()
```

    ## # A tibble: 6 x 3
    ##   energy loudness raw_power
    ##    <dbl>    <dbl>     <dbl>
    ## 1  0.317   -10.7       3.90
    ## 2  0.792    -2.75     16.1 
    ## 3  0.488    -7.05      7.79
    ## 4  0.479    -5.57      8.36
    ## 5  0.73     -3.71     14.1 
    ## 6  0.904    -2.73     18.3

For a bit more on what `mutate()` can do, [check out the nice reference
material](https://dplyr.tidyverse.org/reference/mutate.html).

**Exercise 3**  
Let’s test your knowledge a bit. Create a new variable called `boogie`
that is danceability multiplied by tempo. Call the new data frame
`spotify_boogie`.

-----

## Visualizing trends

Besides visualizing the distributions of individual variables,
visualizing trends between two or more variables is an important
component of exploratory data analysis. There are quite a few options
for visualizing trends among variables, but we will focus on two here-
scatterplots and time series plots.

### Scatterplot

Lets get our hands dirty and make a quick plot of the relationship
between loudness and energy. I have a feeling that they are strongly
positively related. To make a scatterplot, we use the `geom_point()`
function. In addition, we are going to specify two variables in the
aesthetics- loudness on the x-axis and energy on the y-axis.

And the plot is just as I expected\! There is a strong positive
relationship between the two variables.

``` r
spotify %>% 
  ggplot(aes(x = loudness, y = energy)) +
  geom_point()
```

![](04_summarizing-data_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

If we want the trend to be more apparent, we can add a trend line, using
`geom_smooth()`\!

Notice how the line is squiggly. The default for `ggplot` is to use a
flexible model to fit to the data, which can result in overfitting.

``` r
spotify %>% 
  ggplot(aes(x = loudness, y = energy)) +
  geom_point() +
  geom_smooth()
```

    ## `geom_smooth()` using method = 'gam' and formula 'y ~ s(x, bs = "cs")'

![](04_summarizing-data_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

This trend looks quadratic, so lets fit a quadratic curve to the data
and see how that looks\!

We’re going to specify the `method` as “lm” to indicate that we want to
fit a linear model. The formula tells it to use a quadratic fit instead
of purely linear fit.

``` r
spotify %>% 
  ggplot(aes(x = loudness, y = energy)) +
  geom_point() +
  geom_smooth(method = "lm", formula = y ~ x + I(x^2))
```

![](04_summarizing-data_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->

### Time-series

One of my favorite ways to explore data is to visualize trends over
time. Whether it’s understanding the spread of a disease over time, or
investigating the impact of sampling day on your biodiversity survey,
temporal trends are often important components of an analysis.

The `spotify` dataset has weekly Hot 100 Charts from Feb. 16th 2019 to
Feb. 13th 2021. We can therefore visualize trends across a weekly or
coarser scale. If we want to visualize these trends, we need to do some
data wrangling first. Since each week has multiple observations (100
songs for each week), a time-series plot using all of the data for a
variable would look very messy:  
![](04_summarizing-data_files/figure-gfm/unnamed-chunk-14-1.png)<!-- -->

Prior to plotting, we should summarize the data by week to “de-noise”
the trends. That way we can see clear week-by-week trends.

This is a large chunk of code, but it should look familiar to you. Here,
we are using `group_by`, `summarize`, and `mutate` to create a new data
frame that summarizes the variables by week. For all continuous
variables we take their median. It’s often desirable to summarize a
factor by its proportion from the total, so I summarized the `mode`
variable by first counting the major and minor songs per week, then
taking the proportion of minor songs relative to the total number of
songs per week.

``` r
spotify_sum <- spotify %>% 
  group_by(week) %>% 
  summarize(
    valence = median(valence),
    tempo = median(tempo),
    speechiness = median(speechiness),
    instrumentalness = median(instrumentalness),
    liveness = median(liveness),
    loudness = median(loudness),
    acousticness = median(acousticness),
    danceability = median(danceability),
    energy = median(energy),
    major = sum(mode == "major"),
    minor = sum(mode == "minor"),
    duration_ms = median(duration_ms),
    time_since_covid = median(time_since_covid),
    covid_period = max(covid_period)
  ) %>% 
  mutate(
    # get the proportion of minor songs for the week
    prop_minor = minor / (major + minor)
    )
```

Now, to visualize the trends\! Let’s take a look at the temporal trend
in loudness over time. To make a time-series plot, we use the
`geom_line` argument to connect between observations with a line. If you
provide the x-axis with a date, `ggplot` knows to handle this value as a
unit of time, rather than a continuous point.

Interesting\! Looks like there has been a slow decline in loundess over
the last two years. I wonder if this has anything to do with the COVID
pandemic?

``` r
spotify_sum %>% 
  ggplot(aes(x = week, y = loudness)) +
  geom_line()
```

![](04_summarizing-data_files/figure-gfm/unnamed-chunk-16-1.png)<!-- -->

We can investigate that using color\! Let’s color the line according to
the COVID period the week was in. This looks to be the case\!

``` r
spotify_sum %>% 
  ggplot(aes(x = week, y = loudness, color = covid_period)) +
  geom_line() +
  scale_color_viridis_d()
```

![](04_summarizing-data_files/figure-gfm/unnamed-chunk-17-1.png)<!-- -->

Maybe a certain other visualization technique can show this relationship
clearer…

``` r
spotify_sum %>% 
  ggplot(aes(x = loudness, color = covid_period)) +
  geom_density() +
  scale_color_viridis_d()
```

![](04_summarizing-data_files/figure-gfm/unnamed-chunk-18-1.png)<!-- -->
Neat\! It looks like the difference in loudness between pre-COVID songs
and post-COVID songs is much more marked when removing the within-chart
noise from the data.

Rather than give you a specific exercise, I want to encourage you to
explore the new `spotify_sum` data set using the tools we’ve learned so
far. For instance, there appears to be a marked decrease in loudness
around January every year. Why might that be? Use the `filter` function
to find out\! Or, maybe you want to see what trends that we’ve already
explored with the full dataset pop out when using the summarized
dataset. I think you’ll be surprised at the differences\!

## Go further

We’ve barely tipped the iceberg of R’s data visualization capabilities.
For more, check out these fine resources:

  - [R for Data Science](https://r4ds.had.co.nz/data-visualisation.html)
    for a more broad overview of `ggplot2`’s capabilities  
  - The
    [r-spatial](https://www.r-spatial.org/r/2018/10/25/ggplot2-sf.html)
    page for making maps in `ggplot2`

-----

## Recap

Congratulations\! You can use `dplyr`’s grammar of data manipulation to
access any data associated with a table—even if that data is not
currently displayed by the table.

In other words, you now know how to look at data in R, as well as how to
access specific values and calculate summary statistics.

If you’re raring to test out your skill set and expand it, I strongly
recommend going through the [exploratory data analysis
chapter](https://r4ds.had.co.nz/exploratory-data-analysis.html) in R for
Data Science, then reading and coding through the rest of the book if
you feel compelled to (which you probably will).

If you want data sets to play with, I can’t recommend
[TidyTuesday](https://www.tidytuesday.com/) enough\! TidyTuesday is a
weekly podcast and community activity to unite folks who want to improve
their data analysis and visualization skills in R. A common data set is
supplied on the project’s [github
account](https://github.com/rfordatascience/tidytuesday) that members
explore in their preferred manner and share. Some do live code throughs
([also see here](https://www.youtube.com/c/JuliaSilge/featured)) so you
can watch and learn from experts exploring data for the first time\!

A wonderful community (that TidyTuesday happens to be a part of) is the
[r4ds community](https://www.rfordatasci.com/), which provides resources
for learning, mentoring, and staying connected in the R world. Their
Slack channel is a resource that I wish I’d known about earlier in my R
journey- there are many opportunities to pose questions, learn from
other users, and even find out about jobs\! It is also one of those rare
online communities that stays healthy and supportive- beginners and
experts are welcome without judgement.

Lastly, the R community is very supportive and R users strive to
cultivate a positive environment. R has a strong twitter presence
(search for the \#rstats hashtag and start following\!) filled with
generally positive (and helpful\!) posts.

Thanks y’all\!

-----

## Answers

**Exercise 1**

``` r
spotify %>%
  filter(season == "Fall") %>%
  summarize(
    avg_loud = mean(loudness),
    max_dance = max(danceability),
    min_energy = min(energy)
  )
```

    ## # A tibble: 1 x 3
    ##   avg_loud max_dance min_energy
    ##      <dbl>     <dbl>      <dbl>
    ## 1    -6.13     0.974      0.158

**Exercise 2**

``` r
spotify %>% 
  filter(rank <= 40) %>% 
  group_by(season) %>% 
  summarize(mean_dance = mean(danceability))
```

    ## # A tibble: 4 x 2
    ##   season mean_dance
    ## * <chr>       <dbl>
    ## 1 Fall        0.685
    ## 2 Spring      0.705
    ## 3 Summer      0.705
    ## 4 Winter      0.678

**Exercise 3**

``` r
spotify_boogie <- spotify %>% 
  mutate(boogie = danceability * tempo)

# not necessary, but nice to see what the new variable looks like relative to the original variables
spotify_boogie %>% 
  select(danceability, tempo, boogie) %>% 
  head()
```

    ## # A tibble: 6 x 3
    ##   danceability tempo boogie
    ##          <dbl> <dbl>  <dbl>
    ## 1        0.778 140.   109. 
    ## 2        0.687 100.    68.7
    ## 3        0.752 136.   102. 
    ## 4        0.76   89.9   68.3
    ## 5        0.834 155.   129. 
    ## 6        0.579  82.0   47.5

-----

[\<\<\< Previous](03_piping.md)
