# _League of Legends Analysis_
 *This is DSC 80 Project 4 where we clean, explore, and make predictions upon, League of Legends game data. This website stands as a report of our findings.*

**Names:** Rihui Ling and Minghan Wu

## Introduction

Our dataset is on all of the professional League of Legends games between 2014 and 2024 (inclusive).
The dataset contains one row per game per team (so two rows per game). The game data is comprehensive of most aspects of the game. Our cleaned game data has ```131414 rows × 27 columns```.
In this report, we discuss one question to perform hypothesis test on, and one prediction problem.

*Hypotheses:*

In the history of LoL, the choice of side (blue or red) is often believed to have an influence on the team winning rate, with the blue side having a greater chance of winning. For this reason, we put forth this question:

_Will the choice of side by a team in a game affect the team kills?_

* $H_0$: The blue side has the same expected team kills as the red side.

* $H_1$: The blue side has higher expected team kills than the red side.

*Prediction Problem:*
* Predict whether a team is winning/losing a game, given other data.

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

1 We only consider team data, so drop all player rows, only keeping the rows where ```position``` is ```team```
2 Drop all columns that are all na values, which is not associated with teams
3 Drop rows that missing percentage is above the average (2 col), and all columns where all rows are missing
4 Keep only the columns useful for our analysis
5 Convert ```patch``` to major patch; Drop all missing ```patch``` (since ```patch``` is a key important information)
6 Drop all rows where ```gamelength``` is greater than 2 hrs (since the longest game in the history of LOL is about 1h30min), convert the unit of ```gamelength``` from s to min
7 drop rows that the earned gold is less than 0, it should only contain positive value.
8 Convert all binary encoded columns to ```bool```

Here are the columns we decide to keep:

* Basic game stats: ```gameid``` ```patch``` ```gamelength```
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
src="table/col_missing.html" 
width=800
height=600
frameBorder=0>
</iframe>

### Not Missing at Random (NMAR)

We believe the missingness of ```game``` is NMAR. Upon researching online, we discover that the missingness results largely from the fact that there is only one game in the split in that league and year.

As such, the missing mechanism of ```game``` is NMAR, with value 1 (representing the first game) being significantly more prone to be missing than other values.

### Missing at Random (MAR)

We believe the missingness of ```elders``` is dependent on ```patch```. To confirm this observation, we run a permutation test on the two columns.

This is the missing percentage of elder by patch:

<iframe 
src="table/elder_missingness.html" 
width=800
height=600
frameBorder=0>
</iframe>

Visualize observed conditional distribution:

<iframe 
src="img/distribution_of_patch_by_missingness_of_elders.html" 
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

Hence, we conclude that the missingness of ```elders``` is dependent on ```patch```, making the former missing at random.

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

From this, we can see that the missingness of ```gameid``` is independent from the values of other columns, with 99% confidence level.

### Handling Missingness

* Listwise delete all rows where ```gameid``` is missing.
* Use ```patch``` to conditionally impute missing ```elders```. We use ```np.random.choice``` to randomly select $n$ ```elders``` values from the rows with the same ```patch```, where $n$ is the number of ```elders``` missing corresponding to that ```patch```.

## Hypothesis Testing

Question: _Will the choice of side by a team in a game affect the team kills?_

* $H_0$: The blue side has the same expected team kills as the red side.

* $H_1$: The blue side has higher expected team kills than the red side.

Here is a graph showing the overlaying distributions of red-side kills and blue-side kills:

<iframe 
src="img/distribution_of_kills_by_side.html" 
width=800
height=600
frameBorder=0>
</iframe>

We test the two hypotheses by conducting a permutation test by permuting ```kills``` 1000 times and compute the test statistic $average\_kills\_blue - average\_kills\_red$.

Below is a graph showing the distribution of the test tatistic according to $H_0$ and our observed statistic:

<iframe 
src="img/distribution_of_difference_in_mean_(kills).html" 
width=800
height=600
frameBorder=0>
</iframe>

Since p-value we obtain is 0.0, we reject $H_0$. This test makes us conclude that more likely that not, the choosing the blue side does increase the expected kills of the team.

## Framing a Prediction Problem

We try to obtain a model that predicts whether a team is winning/losing a game, given the data we kept.

This is a binary classification problem.

We will use (test set) accuracy to evaluate our model, since 
* It is a more direct metric for the success of the model
* We do not have a preference towards false positive / false negative
* The ```result``` we get after cleaning and imputation has about equal numbers of ```1``` and ```0``` (the percentage of winning is 50.01%)
