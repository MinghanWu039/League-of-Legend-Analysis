# League of Legends Analysis
 *This is DSC 80 Project 4 where we clean, explore, and make predictions upon, League of Legends game data. This website stands as a report of our findings.*

**Names:** Rihui Ling and Minghan Wu

## Introduction

Our dataset is on all of the professional League of Legends games between 2014 and 2024 (inclusive).
The dataset contains one row per game per team (so two rows per game). The game data is comprehensive of most aspects of the game. Our cleaned game data has ```132058 rows × 30 columns```.
In this report, we discuss one question to perform hypothesis test on, and one prediction problem.

*Hypotheses:*
* H0: The average kills per game in LPL is equal to the average kills in all leagues.

* H1:The average kills per game in LPL is greater than the average kills in all leagues.

*Prediction Problem:*
* Predict whether the team is winning/losing given all other aforementioned data.

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

1 We only consider team data, so drop all player rows, only keeping the rows where ```position``` is ```team```.
2 Drop all columns that are all na values, which is not associated with teams
3 Drop rows that missing percentage is above the average (2 col), and all columns where all rows are missing.
4 Keep only the columns useful for our analysis
5 Replace the unknown team in teamname as na value
6 Convert ```patch``` to major patch
7 Drop all rows where ```gamelength``` is greater than 2 hrs (since the longest game in the history of LOL is about 1h30min), convert the unit of ```gamelength``` from s to min.
8 drop rows that the earned gold is less than 0, it should only contain positive value.
9 Convert all binary encoded columns to ```bool```.

Here are the columns we decide to keep:

* Basic game stats: ```gameid``` ```league``` ```year``` ```game``` ```patch``` ```gamelength```
* Basic team stats: ```side``` ```teamname``` ```teamid```  
* Team game results: ```result``` ```kills``` ```deaths``` ```assists``` ```firstblood``` ```damagetochampions``` ```towers``` ```opp_towers``` ```firstmidtower``` ```firsttothreetowers```
* General game resources: ```earnedgold``` ```firstdragon``` ```dragons``` ```opp_dragons``` ```firstherald``` ```elders``` ```opp_elders``` ```firstbaron``` ```barons``` ```opp_barons``` ```opp_barons``` 

### Univariate Analysis

Here is a histogram showing the distribution of ```gamelength```:

<iframe 
src="img/distribution_of_game_length_(min).html" 
width=800 
height=600
frameBorder=0>
</iframe>

We can see that the data is roughly normally distributed, centering around 32.5 min. Note that there are outliers on both ends of the graph (even after we remove the extreme values), although their density is too small to be visible.

### Bivariate Analysis

Here is a graph showing the distributions of gold earned by the team in the game conditional on game results (```win``` or ```lose```):

<iframe 
src="img/distribution_of_earned_gold_each_game_by_result.html" 
width=800
height=600
frameBorder=0>
</iframe>

Both distributions are roughly normally distributed, with the ```win``` distribution more concentrated than the ```lose``` distribution, and with higher mean.

### Interesting Aggregates

The following pivot table shows the average team kills for lost and won teams for different years.

<iframe 
src="table/avg_kills.html" 
width=800 
height=600
frameBorder=0>
</iframe>

We can see that for each year, the difference between average team kills for won teams and lost teams are around 10. Also, the average team kills for both won teams and lost teams increased significantly since 2021.

## Missingness Mechanisms

In this section, we analyze the missing mechanisms of several of the columns. To give an overview, here is a DataFrame showing how many rows are missing for each column:

<iframe 
src="table/rows_missing_by_columns.html" 
width=800
height=600
frameBorder=0>
</iframe>

### Missing at Random (MAR)

We believe the missingness of ```elders``` is dependent on ```patch```. To confirm this observation, we run a permutation test on the two columns.

This is the observed conditional distribution:

<iframe 
src="img/distribution_of_patch_by_missingness_of_elders.html" 
width=800
height=600
frameBorder=0>
</iframe>

This is the DataFrame showing the same distribution:

<iframe 
src="table/patch_missingness.html" 
width=800
height=600
frameBorder=0>
</iframe>

After running the permutation and computing Total Variation Distance (TVD) 500 times, this is the distribution we get:

<iframe 
src="img/permutation_test_of_the_missingness_of_elders_depend_on_patch.html" 
width=800
height=600
frameBorder=0>
</iframe>

We see that the observed tvd 1.2765 is significantly greater than most of the tvd values resulted from distribution. The p-value for the permutation test is 0.0. 

Hence we conclude that the missingness of ```elders``` is dependent on ```patch```, making the former missing at random.

### Missing Completely at Random (MCAR)

We believe the missing mechanism of ```gameid``` is completely at random. To confirm this, we run permutation tests against each every one of other columns. 

For numerical columns, we compute the absolute differences between the missing group and the non-missing group; 

For categorical columns, we compute the tvds between the missing group and the non-missing group.

Here is the DataFrame showing our result:

<iframe 
src="table/gameid_missingness.html" 
width=800
height=600
frameBorder=0>
</iframe>
