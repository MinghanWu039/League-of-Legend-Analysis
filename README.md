<script type="text/javascript" async="" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.7/MathJax.js?config=TeX-MML-AM_CHTML"></script>

# League of Legends
## _Analysis for Game Result Prediction_
 *This is DSC 80 Project 4 where we clean, explore, and make predictions upon, League of Legends game data. This website stands as a report of our findings.*

**Names:** Rihui Ling and Minghan Wu


## Introduction

Our dataset is on all of the professional League of Legends games between 2014 and 2024 (inclusive).
The dataset contains one row per game per team (so two rows per game). The 
game data is comprehensive of most aspects of the game. Our cleaned game data has ```148989 rows × 26 columns```.
In this report, we discuss one question, and one prediction problem.

*Question to Answer:*

In the history of LoL, the choice of side (blue or red) is often believed to have an influence on the team winning rate, with the blue side having a greater chance of winning. If this is true, then the result of game largely depends on luck -- the team on the blue side has a significant advantage -- and hence the outcome of one game does not necessarily reflect the strength of a team. For this reason, we put forth this question:

_Will the choice of side by a team in a game affect the team kills?_

* $$H_0$$: The blue side has the same expected team kills as the red side.

* $$H_1$$: The blue side has higher expected team kills than the red side.

After that, we will come up with model(s) for the following prediction problem:

*Prediction Problem*
* Predict whether a team is winning/losing a game, given other data.

While much of the data we have is only available post game, such a model can indicate what factors are important for winning a LoL game.

### Data

Our raw data has ```897996 rows × 130 columns```. Most of the columns are of little or no help for our analysis. 

Here lists the columns we decide to keep along with their description:

* Basic game stats: 
    * ```gameid```: the game id
    * ```game```: the ordinal number of the game in the competition
    * ```patch``` the major patch number
    * ```gamelength``` the time span of the game in minutes
* Team game stats: 
    * ```side```: the side of the team, blue or red
    * ```result```: result of the team in the game
    * ```kills```: the total kills of the team in the game
    * ```deaths```: the total deaths of the team in the game
    * ```assists```: the total assists of the team in the game
    * ```firstblood```: whether the team got first blood
    * ```damagetochampions```: the total damage made to the enemy team
    * ```earnedgold```: the total number of gold earned by the team
    * ```firsttower```: whether the team got the first tower
    * ```towers```: the number of towers got by the team
    * ```opp_towers```: the number of towers got by the enemy team
    * ```firstmidtower```: whether the team got the first middle tower
    * ```firsttothreetowers```: whether the team got three towers before the enemy team
* General game resources: 
    * ```firstdragon```: whether the team got the first dragon
    * ```dragons```: the total number of dragons got by the team
    * ```opp_dragons```: the total number of dragons got by the enemy team
    * ```firstherald```: whether the team got the first herald
    * ```elders```: the total number of elders got by the team
    * ```opp_elders```: the total number of elders got by the enemy team
    * ```firstbaron```: whether the team got the first baron
    * ```barons```: the total number of barons got by the team
    * ```opp_barons```: the total number of barons got by the enemy team

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

_1_ We only consider team data, so drop all player rows, only keeping the rows where ```position``` is ```team```

_2_ Drop rows that the earned gold is less than or equal to 0, since it should only contain non-negative value and value 0 indicates very short game length.

_3_ Convert ```patch``` to major patch

_4_ Drop all rows where ```gamelength``` is greater than 2 hrs (since the longest game in the history of LOL is about 1h30min), convert the unit of ```gamelength``` from s to min

_5_ Keep only the columns useful for our analysis

_6_ Convert all binary encoded columns to ```bool```

_7_ Convert all numerical column with integral values to ```int```

We follow these steps so that only data that is reasonable and helpful for our analysis is kept. This makes the following analysis easier and less influenced by irrelevant information.

This table shows the first several rows of the dataframe after data cleaning:

<iframe 
src="table/teams_cleaned.html" 
width=750
height=220
frameBorder=0>
</iframe>  
** _Cleaned DataFrame Shape: ```148989 rows × 26 columns```_

### Univariate Analysis

Here is a histogram showing the distribution of ```gamelength```:

<iframe 
src="img/distribution_of_game_length_(min).html" 
width=800 
height=500
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
width=750 
height=400
frameBorder=0>
</iframe>

We can see that for each year, the difference between average team kills for won teams and lost teams are around 10 (so ```kills``` and ```result``` are strongly related). Also, the average team kills for both won teams and lost teams increased significantly since 2021.

## Missingness Mechanisms

In this section, we analyze the missing mechanisms of several of the columns. To give an overview, here is a DataFrame showing how many rows are missing for each column:

<iframe 
src="table/col_missing.html" 
width=750
height=250
frameBorder=0>
</iframe>

### Not Missing at Random (NMAR)

We believe the missingness of ```game``` is NMAR. Upon researching online, we discover that the missingness results largely from the fact that there is only one game in the split in that league and year.

As such, the missing mechanism of ```game``` is NMAR, with value 1 (representing the first game) being significantly more prone to be missing than other values.

### Missing at Random (MAR)

We believe the missingness of ```elders``` and ```opp_elders```is not NMAR because there is no reason why the missingness of ```elders``` is dependent on the (missing) values themselves.

However, we believe the missingness of ```elders``` and ```opp_elders``` is dependent on ```patch```. To confirm this idea, we run a permutation test on the two columns (using Total Variation Distance (tvd) as statistic).

This is the observed conditional distribution:

<iframe 
src="img/distribution_of_patch_by_missingness_of_elders.html" 
width=800
height=500
frameBorder=0>
</iframe>

This is the DataFrame showing the same distribution:

<iframe 
src="table/elder_missingness.html" 
width=750
height=450
frameBorder=0>
</iframe>

After running the permutation and computing Total Variation Distance (TVD) 500 times, this is the distribution we get:

<iframe 
src="img/permutation_test_of_the_missingness_of_elders_depend_on_patch.html" 
width=800
height=500
frameBorder=0>
</iframe>

We see that the observed tvd $$0.615131$$ is significantly greater than all tvd values from the distribution. The _p-value_ for the permutation 
test is $$0.0$$. 

Hence, we conclude that the missingness of ```elders``` is dependent on ```patch```, making the former missing at random.

After running the same test on the missingness of ```opp_elders```, we find the same pattern (which is expected because it is inherently in pairs with ```elders```)


### Missing Completely at Random (MCAR)

We believe the missing mechanism of ```barons``` and ```opp_barons``` is completely at random. To confirm this, we run permutation tests against each every one of other columns. 

For numerical columns, we compute the absolute differences between the means of the missing group and the non-missing group; 

For categorical columns, we compute the tvds between the missing group and the non-missing group.

Here is the DataFrame showing our result for ```barons```:

<iframe 
src="table/barons_missingness.html" 
width=750
height=400
frameBorder=0>
</iframe>

From this, we can see that the missingness of ```barons``` is independent from the values of other columns, with 95% confidence level. 

(Some p-values are close to 1, this is because the number of missing ```barons``` is very few.)

After running the same test on the missingness of ```opp_barons```, we got the same result.

### Handling Missingness (Imputation)

* Perform probabilistic imputation on missing ```elders``` conditional on ```patch```. We use ```np.random.choice``` to randomly select $$n$$ ```elders``` values from the rows with the same ```patch```, where $$n$$ is the number of ```elders``` missing corresponding to that ```patch``` (we regard na value in ```patch``` as a valid category). Note that for those patches where ```elders``` do not exist, the non-missing values are all 0, and hence the imputed values must be 0.

* Listwise delete rows where ```barons``` is missing (delete all rows where ```barons``` is missing), since ```barons``` is missing completely at random.

## Hypothesis Testing

As mentioned in the introduction, in the history of LoL, the choice of side 
(blue or red) is often believed to have an influence on the team winning rate, 
with the blue side having a greater chance of winning. In this section, we 
want to test out will the kills, which is high correlated with win rate, for 
each side is having significant difference.

Question: _Will the of side by a team in a game affect the team kills?_

* $$H_0$$: The blue side has the same expected team kills as the red side.

* $$H_1$$: The blue side has higher expected team kills than the red side.

* Test statistic: _average kills blue - average kills red_

* Significance level: 5%

Here is a graph showing the overlaying distributions of red-side kills and blue-side kills:

<iframe 
src="img/distribution_of_kills_by_side.html" 
width=800
height=600
frameBorder=0>
</iframe>

This dataframe shows the observed data:

<iframe 
src="table/kills_observed.html" 
width=750
height=150
frameBorder=0>
</iframe>

We test the two hypotheses by conducting a permutation test by permuting ```kills``` 1000 times and compute the test statistic .

Below is a graph showing the distribution of the test tatistic according to $$H_0$$ and our observed statistic.

<iframe 
src="img/distribution_of_difference_in_mean_(kills).html" 
width=800
height=500
frameBorder=0>
</iframe>

Since _p-value_ we obtain is 0.0 and $$0.0<0.05$$, we reject $$H_0$$. This test makes us 
conclude that more likely than not, the team being on the blue side does increase the expected kills of the team.

## Framing a Prediction Problem

Prediction Problem: We try to obtain a model that predicts whether a team is winning/losing a game, given the data we kept.

This is a binary classification problem.

We will use (test set) accuracy to evaluate our model, since 
* It is a more direct metric for the success of the model
* We do not have a preference towards false positive / false negative
* The ```result``` we get after cleaning and imputation has about equal 
  numbers of ```True``` and ```False``` (the percentage of _winning_ is $$50.01%$$)

## Baseline Model

Our baseline model uses _logistic regression_ to predict whether the team will win or lose given the 22 features we select. 

Here is a dataframe showing the features we use in our baseline model, separated by category (nominal, ordinal, numerical). 

For this model, all features come from the cleaned dataframe after imputation.

<iframe 
src="table/model_df.html" 
width=750 
height=280
frameBorder=0>
</iframe>

Here, we describe what transformation we have applied to each category:

**Nominal**

We have 8 nominal columns. 

We one-hot-encode every categorical column (each time, drop one of the columns generated to avoid colinearity).

**Numerical**

We have 14 numerical columns.

We standardize every numerical column.

That is, for every column $$X=(X_1, X_2, ..., X_n)$$, for every $$i=1, 2, ..., n$$, we let $$X_{std, i}=\frac{X_i-\bar{X}}{SD_X}$$.

**Ordinal**

_We do not have any ordinal feature in our selection, since no categorical column has more than 2 values that have an inherent order i.e. exchanging the labels makes no difference._


### Model Parameters

For this model, we use default parameters of sklearn.LogisticRegression.

* Penalty: ```'l2'```
* tol (tolerance for stopping iteration): ```1e-4```
* max_iter: ```100```
* solver: ```lbfgs```

### Model Result

The following dataframe shows the model accuracy:

<iframe 
src="table/baseline_model_score.html" 
width=750 
height=100
frameBorder=0>
</iframe>

The following graph is the confusion matrix on the test set:

<iframe 
src="img/confusion_matrix_of_test_data_(baseline_model).html" 
width=750
height=500
frameBorder=0>
</iframe>

We believe this model is effective, based on the high accuracy it gets.

## Final Model

In this model, we have added derivative features to the dataframe, together with the original 22 features. The final model has 30 features in total.

* ```kda```: Calculated by $$\frac{kills+0.5\times assists}{deaths}$$ (If for a row```death``` is 0, we set ```death``` to 1). This is a commonly used measure for team performance. It takes into account the ratio of kills and assists to deaths.
* ```soul``` ```opp_soul```: Binary indicator of whether 
  ```dragons```/```opp_dragons``` is greater than or equal to 4. This is important because in later 
  patch, a team gets dragon soul buff once they get 4 dragons.
* ```kills_per_min``` ```deaths_per_min``` ```assists_per_min``` ```damagetochampions_per_min``` ```earnedgold_per_min```: Those columns divided by ```gamelength```. This can be helpful because they indicate the rate at which the data changes in the game.

For the final model, we decide to stick to logistic regression. 

In order for the model to be fairly comparable to the baseline model, we use the same training set and test set as before.

### Best Hypermeters

We want to optimize the following hypermeters:

* ```max_iter``` 
    > Maximum number of iterations taken for the solvers to converge.
    * Range: $$\{100, 150, 250, 300\}$$
    
* ```solver```
    > Algorithm to use in the optimization problem.
    * Range: \{"lbfgs", "libliear", "newton-cg", "sag", "saga"\}

Since ```tol``` (Tolerance for stopping criteria) is highly related to 
```max_iter```, we do not optimize it. Instead, we keep it as default 
$$10^{-4}$$ 
(or ```1e^-4```).

To determine the best combination, we use GridSearchCV to perform a grid search (comparing all combinations of ```max_iter``` and ```solver```) with 5-fold cross-validations on the training set.

This is the result of GridSearchCV:

<iframe 
src="table/hypermeters.html" 
width=750
height=500
frameBorder=0>
</iframe>

From this process, we conclude that the best parameter combination is

* ```max_iter = 100```

* ```solver = "newton-cg"```.

Here shows the confusion matrix on the test set of the optimal model that we found:

<iframe 
src="img/confusion_matrix_of_test_data_(final_model).html" 
width=750
height=500
frameBorder=0>
</iframe>

Here the two confusion matrices are shown side by side:

<iframe 
src="img/model_matrixs.html" 
width=800
height=450
frameBorder=0>
</iframe>

Here is a summary of the model accuracy of the baseline model and the final model. There is a slight improvement compared to the baseline model.

<iframe 
src="table/model_scores.html" 
width=750
height=180
frameBorder=0>
</iframe>

## Fairness Analysis

Could our model perform worse for blue or red side due to the fact that they start in slightly different positions?

In this section, we want to explore the consistency of the performance of 
our model for red and blue sides. We want to find out whether the final model 
achieve accuracy parity for blue and red sides. i.e. is the difference in 
accuracy due to randomness?

_Two Groups:_ blue side and red side

$$H_0$$: Our model is fair. That is, the accuracy of the final model for blue side is the same as the accuracy for red side.

$$H_1$$: The accuracy of the final model for blue side is smaller than the 
accuracy for the red side.

_Test Statistic:_ $$accuracy(blue)-accuracy(red)$$

_Significance Level_: 5%

_Evaluation Metric_: accuracy

We run a permutation test to determine which hypothesis is correct:

We compute the observed difference, and simulate the $$H_0$$ distribution by permuting ```side``` 1000 times and compute the difference between the accuracy for "blue" side and the accuracy for "red" side.

This dataframe shows the observed statistic:

<iframe 
src="table/fairness_observed.html" 
width=750
height=150
frameBorder=0>
</iframe>

This graph shows the result of the permutation test:

<iframe 
src="img/distribution_of_difference_in_accuracy.html" 
width=800
height=500
frameBorder=0>
</iframe>

Since the _p-value_ is 0.294 and $$0.294>0.05$$, we fail to reject $$H_0$$ at 
5% significance level. The difference in observed accuracy is not statistically significant to suggest that our model works better on blue side than on red side.

