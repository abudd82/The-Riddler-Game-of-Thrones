How Many Soldiers Do You Need To Beat The Night King?
================
Alan Budd
5/17/2019

# Introduction

[The May 17th, 2019 Riddler
Classic](https://fivethirtyeight.com/features/how-many-soldiers-do-you-need-to-beat-the-night-king/)
from [FiveThirtyEight](FiveThirtyEight.com)’s Oliver Roeder asks how big
the armies for the living & the dead from HBO’s Game of Thrones need to
be to give each army a 50-50 chance of winning:

> Forget the Battle of Winterfell and model our battle as follows. Each
> army lines up single file, facing the other army. One soldier steps
> forward from each line and the pair duels — half the time the living
> soldier wins, half the time the dead soldier wins. If the living
> soldier wins, he goes to the back of his army’s line, and the dead
> soldier is out (the living army uses dragonglass weapons, so the dead
> soldier is dead forever this time). If the dead soldier wins, he goes
> to the back of their army’s line, but this time the (formerly) living
> soldier joins him there. (Reanimation is instantaneous for this Night
> King.) The battle continues until one army is entirely eliminated.

> What starting sizes of the armies, living and dead, give each army a
> 50-50 chance of winning?

# Methodology

In the spirit of FiveThirtyEight’s favorite methodology for forecasting,
I’ll attempt to answer this question using repeated simulations. The
goal by the end of this document will be to develop a function that can
be repeated to find the probability of winning across different
combinations of army sizes. Once that’s computed, we can find what
combination leads to a 50/50 probability.

# Code

Before developing a function, it’s probably worth developing code for
one simulation, i.e. given the conditions set above, and with 2
different army sizes, which side wins a specific simulation. From here
on out, I will use the term “simulation” and “war” interchangably, since
one simulation represents one war, composed of however many “battles”
until an army has no more soldiers.

For this example, let’s assume that both armies have an equal size of
100 soldiers each (we can safely assume that in these scenarios, the
dead army would have a clear advantage, and we should see that
represented in the simulated probability)

``` r
living_size <- 100
dead_size <- 100
```

We are assuming that each battle has a 50/50 outcome of either the
living or dead soldier winning. So to simulate one battle, we can just
randomly sample from a list containing both “living” and “dead”.

``` r
outcome_list <- c("Living", "Dead")

outcome <- sample(outcome_list, 1)
```

Once the outcome of the individual battle is randomly set, we can apply
the rule listed in the problem statement regarding what happens if
either side wins.

``` r
if(outcome == "Living") {
  dead_size <- dead_size - 1
} else {
  living_size <- living_size - 1
  dead_size <- dead_size + 1
}

living_size
```

    ## [1] 99

``` r
dead_size
```

    ## [1] 101

Now that we have the initial size of the armies, the process for
deciding the winner of a battle, and the rules for what happens after
the battle is decided, we can put all of this together into a function
that simulates a war until an army is out of soldiers.

``` r
war <- function(living_size, dead_size){
  outcome_list <- c("Living", "Dead")
  num_battles <- 0
  
  while(living_size > 0 && dead_size > 0) {
    outcome <- sample(outcome_list, 1)
    if(outcome == "Living"){
      dead_size <- dead_size - 1
    } else {
        living_size <- living_size - 1
        dead_size <- dead_size + 1
    }
    num_battles <- num_battles + 1
  }
  if(living_size == 0){
    winner <- "Dead"
  } else {
    winner <- "Living"
  }
  return(winner)
}
```

Let’s go to war\!

``` r
war_outcome <- war(100, 100)

paste("The Army of the", war_outcome, "won the war.")
```

    ## [1] "The Army of the Dead won the war."

Now that we have the framework for an individual war, we want to repeat
the war under the same conditions and collect the results of each war to
determine the simulated probabilities of each side winning the war given
the starting army size.

``` r
simulate_wars <- function(living_size, dead_size, num_simulations){
  living_wins <- 0
  dead_wins <- 0
  for(i in 1:num_simulations){
    war_outcome <- war(living_size, dead_size)
    
    if(war_outcome == "Living"){
      living_wins <- living_wins + 1
    } else {
      dead_wins <- dead_wins + 1
    }
  }
  
  return(list("living_wins" = living_wins,
              "dead_wins" = dead_wins,
              "total_wars" = num_simulations,
              "living_win_pct" = living_wins/num_simulations,
              "living_relative_size" = living_size/dead_size))
}
```

Now that our overall war simulation function is finalized, let’s
simulate 10,000 wars where each army starts with 100
soldiers.

``` r
example1 <- simulate_wars(living_size = 100, dead_size = 100, num_simulations = 1000)

paste("The Army of the Living won", scales::percent(example1$living_wins/example1$total_wars), "of the simulated wars.")
```

    ## [1] "The Army of the Living won 0% of the simulated wars."

``` r
paste("The Army of the Dead won", scales::percent(example1$dead_wins/example1$total_wars), "of the simulated wars.")
```

    ## [1] "The Army of the Dead won 100% of the simulated wars."

Not looking good for the living in this scenario. What about if the
living have a 10-1
advantage?

``` r
example2 <- simulate_wars(living_size = 1000, dead_size = 100, num_simulations = 1000)

paste("The Army of the Living won", scales::percent(example2$living_wins/example2$total_wars), "of the simulated wars.")
```

    ## [1] "The Army of the Living won 2.70% of the simulated wars."

``` r
paste("The Army of the Dead won", scales::percent(example2$dead_wins/example2$total_wars), "of the simulated wars.")
```

    ## [1] "The Army of the Dead won 97.3% of the simulated wars."

Still not looking good, but better than 0%\!

# Answer

Now that all of our functions have been created, we’re now prepared to
answer the original question from The Riddler: what army sizes, both
living and dead, provide a 50-50 chance of winning?

To answer this using our functions above, we’ll cycle through different
sizes of armies for the *living*, while keeping the army size of the
*dead* fixed at 100. We’ll start with a living army size of 100 (which
should yeild a near-zero probability) and progress upwards.

``` r
simulate_sizes <- function(max_living_size, num_sims){
loop_seq <- seq(from = 100, to = max_living_size, by = 100)
results_df <- data.frame()

for(i in loop_seq){
  sim <- simulate_wars(living_size = i, dead_size = 100, num_simulations = num_sims)
  
  results_df <- rbind(results_df, as.data.frame(sim))
}

return(results_df)
}
```

``` r
results_df <- simulate_sizes(max_living_size = 15000, num_sims = 10000)
```

Now that we have a data.frame of different win probabilities based on
different living army sizes, we can easily visualize where the threshold
is for the tipping point.

*Note: Running the above chunk of code took about 17 hours on my Mac. A
future optimization could be to leverage parellel processing.*

``` r
line1_df <- data.frame(x1 = 110, y1 = 0, x2 = 110, y2 = .5)
line2_df <- data.frame(x1 = 0, y1 = .5, x2 = 110, y2 = .5)

ggplot(data = results_df, aes(x = living_relative_size, y = living_win_pct)) +
  geom_line() +
  geom_segment(aes(x = x1, y = y1, xend = x2, yend = y2), color = "red", linetype = 2, data = line1_df) +
  geom_segment(aes(x = x1, y = y1, xend = x2, yend = y2), color = "red", linetype = 2, data = line2_df) +
  scale_y_continuous(labels = scales::percent_format(accuracy = 1)) +
  labs(title = "How much bigger does the Army of the Living need to be?",
       subtitle = "Army of the Dead size fixed at 100",
       x = "Army of the Living Size, relative to the Army of the Dead",
       y = "Army of the Living's chance of winning") +
  theme_minimal()
```

![](GameOfThrones_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

Based on the data from the simulations, it looks like it took about a
**110-1** soldier advantage for the Army of the Living to even the odds
against the Army of the Dead.
