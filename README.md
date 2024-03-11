# League of Legends Analysis
 *This is DSC 80 Project 4 where we clean, explore, and make predictions upon, League of Legends game data. This website stands as a report of our findings.*

**Names:** Rihui Ling and Minghan Wu

## Introduction

Our dataset is on all of the professional League of Legends games between 2014 and 2024 (inclusive).
The dataset contains one row per game per team (so two rows per game). The game data is comprehensive of most aspects of the game. Our cleaned game data has ```149384 rows × 114 columns```.
In this report, we discuss one question to perform hypothesis test on, and one prediction problem.

*Hypotheses:*
* H0: The average kills per game in LPL is equal to the average kills in all leagues.

* H1:The average kills per game in LPL is greater than the average kills in all leagues.

*Prediction Problem:*
* Predict whether the team is winning/losing given all other aforementioned data.

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

* We only consider team data, so drop all player rows, only keeping the rows where ```position``` is ```team```.
* dorp all columns that are all na values, which is not associated with teams
* Drop all rows where all columns are missing, and all columns where all rows are missing.
* Convert the ```date``` columns to ```pd.Datetime```.
* Replace the unknown team in teamname as na value
* Drop all rows where ```gamelength``` is greater than 2 hrs (since the longest game in the history of LOL is about 1h30min), convert the unit of ```gamelength``` from s to min.
* drop rows that the earned gold is less than 0, it should only contain positive value.
* Convert all binary encoded columns to ```bool```.

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
src="img/distribution_of_earned_gold_each_game_condition_on_result.html" 
width=800
height=600
frameBorder=0>
</iframe>

Both distributions are roughly normally distributed, with the ```win``` distribution more concentrated than the ```lose``` distribution, and with higher mean.

### Interesting Aggregates

The following pivot table shows the average team kills for lost and won teams for different years.

<iframe 
src="avg_kills.html" 
width=800 
height=600
frameBorder=0>
</iframe>

We can see that for each year, the difference between average team kills for won teams and lost teams are around 10. Also, the average team kills for both won teams and lost teams increased significantly since 2021.