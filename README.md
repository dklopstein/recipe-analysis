# Do healtier recipes have to taste bad?: Investigating the Relationship Between Healthy Recipes and their Rating

Author: Derek Klopstein

---


## Introduction

Food is essential to life. However, that doesn't mean we can eat whatever we want and feel no repercussions. While there may be no immediate changes for eating a diet consisting of sugary substances, the long term effects will certainly hurt you. In our current generation, gym culture and eating more healthy and promoted, and is very popular. However, people who don't follow media or aren't aware of such trends might not feel encouraged to eat more healthy, especially if they think healthy foods taste worse. **I wondered if people would rate a healthy recipe lower based on their preconceived perception that "healthier" ingredients taste worse.** To answer this question, I was given two dataset consisting of recipes and user interations since 2008 on [food.com](food.com).

The first dataset, `recipe`, contains 83782 rows. Each row is a unique recipe and there are 10 columns:

| Column             | Description                                                                                                                                                                                       |
| :----------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `'name'`           | Recipe name                                                                                                                                                                                       |
| `'id'`             | Recipe ID                                                                                                                                                                                         |
| `'minutes'`        | Minutes to prepare recipe                                                                                                                                                                         |
| `'contributor_id'` | User ID who submitted this recipe                                                                                                                                                                 |
| `'submitted'`      | Date recipe was submitted                                                                                                                                                                         |
| `'tags'`           | Food.com tags for recipe                                                                                                                                                                          |
| `'nutrition'`      | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value” |
| `'n_steps'`        | Number of steps in recipe                                                                                                                                                                         |
| `'steps'`          | Text for recipe steps, in order                                                                                                                                                                   |
| `'description'`    | User-provided description                                                                                                                                                                         |
| `'ingredients'`    | Text for recipe ingredients                                                                                                                                                                       |
| `'n_ingredients'`  | Number of ingredients in recipe                                                                                                                                                                   |

The second dataset, `interactions`, contains 731927 rows. Each row contains a user interaction with a specific recipe and there are 5 columns:

| Column        | Description         |
| :------------ | :------------------ |
| `'user_id'`   | User ID             |
| `'recipe_id'` | Recipe ID           |
| `'date'`      | Date of interaction |
| `'rating'`    | Rating given        |
| `'review'`    | Review text         |

**Given the datasets, I am investigating whether people rate healthy recipes and the non-healthy recipes on the same scale.** By seeking an answer to our question, we would have an insight on people’s preference on generally healthier recipes, which could help contributors on Food.com promote healthier lifestyles. 

## Data Cleaning and Exploratory Data Analysis

To make the analysis of the dataset more efficient, I did the following for the data cleaning steps.

1. Left merge the recipes and interactions datasets on `'id'` and `'recipe_id'`.

   - This step matches the unique recipes with their user interactions from the relevant dataset.
  
1. Split values in the `'nutrition'` column to individual columns of floats.

   - Even though the values in the nutrition column look like a list, they are actually objects, which act like strings. Given by the description of the columns of the recipe dataset, we know what each individual values inside the brackets mean. To split up the values, we applied a lambda function then converted the columns to floats. It will allow us to conduct numerical calculations on the columns.

1. Convert `'submitted'` and `'date'` to datetime.

   - These two columns are both stored as objects initially, so we converted them into datetime to allow us conduct analysis on trends over time if needed.

1. Fill all ratings of 0 with np.NaN.

   - A rating of 0 indicates missing values in rating. We know this because ratings of 0 signify that the user has yet to make the recipe, as all recipes are generally given a rating on a 1-5 scale, 1 being the lowest. Therefore, to avoid bias in the ratings, we filled the value 0 with np.nan.

1. Convert `'ingredients'` and `'tags'` to lists and add `'num_tags'` column.

  - This allows us to see the most common ingredients and tags within all the recipes.

1. Add column `'average_rating'` containing average rating per recipe.

   - Since a recipe can have numerous ratings from different users, we take an average of all the ratings to get a more comprehensive understanding of the rating of a given recipe.


1. Add `'is_healthy'` to the dataframe

   - `'is_healthy'` is a boolean column checking if the tags of recipes contain 'healthy,' which separates the recipes into two groups, ones that are healthy and ones that are not. This provides us a way to compare the other columns with healthy and non-healthy recipes.

#### Result
Here are all the columns of the cleaned df.

| Column                  | Description    |
| :---------------------- | :------------- |
| `'name'`                | object         |
| `'id'`                  | int64          |
| `'minutes'`             | int64          |
| `'contributor_id'`      | int64          |
| `'submitted'`           | datetime64[ns] |
| `'tags'`                | object         |
| `'nutrition'`           | object         |
| `'n_steps'`             | int64          |
| `'steps'`               | object         |
| `'description'`         | object         |
| `'ingredients'`         | object         |
| `'n_ingredients'`       | int64          |
| `'user_id'`             | float64        |
| `'recipe_id'`           | float64        |
| `'date'`                | datetime64[ns] |
| `'rating'`              | float64        |
| `'review'`              | object         |
| `'calories (#)'`        | float64        |
| `'total fat (PDV)'`     | float64        |
| `sugar (PDV)'`          | float64        |
| `'sodium (PDV)'`        | float64        |
| `'protein (PDV)'`       | float64        |
| `'saturated fat (PDV)'` | float64        |
| `'carbohydrates (PDV)'` | float64        |
| `'avg_rating'`          | float64        |
| `'is_healthy'`          | bool           |


Our cleaned dataframe ended up with 234429 rows and 27 columns. Here are the first 5 rows of ~unique recipes of our cleaned dataframe. Since there is a lot of columns for the merged dataframe, we selected the columns that are most relevant to our questions for display. Scroll right to view more columns.

| name                                 |     id |   minutes | submitted           |   rating |       avg_rating |   calories (#) |  calories (#) | is_healthy   |
|:-------------------------------------|-------:|----------:|:--------------------|---------:|-----------------:|---------------:|--------------:|:-------------|
| 1 brownies in the world    best ever | 333281 |        40 | 2008-10-27 00:00:00 |        4 |                4 |          138.4 |            50 | False        |
| 1 in canada chocolate chip cookies   | 453467 |        45 | 2011-04-11 00:00:00 |        5 |                5 |          595.1 |           211 | False        |
| 412 broccoli casserole               | 306168 |        40 | 2008-05-30 00:00:00 |        5 |                5 |          194.8 |             6 | False        |
| 412 broccoli casserole               | 306168 |        40 | 2008-05-30 00:00:00 |        5 |                5 |          194.8 |             6 | False        |
| 412 broccoli casserole               | 306168 |        40 | 2008-05-30 00:00:00 |        5 |                5 |          194.8 |             6 | False        |


### Univariate Analysis

For this analysis, I examined the distribution of healthy recipes. As the plot below shows, the distribution skewed to the right, indicating that most of the recipes on food.com are generally given a rating of 5.

<iframe
  src="assets/univariate.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Bivariate Analysis

For this analysis, I examined the distribution of the rating of the recipe conditioned between the healthy recipes and non-healthy recipes. The graph below shows that healthy recipes and non-healthy recipes have similar distrivutions of ratings, but I'll get to their differences later in the analysis.

<iframe
  src="assets/bivariate.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Interesting Aggregates

For this section, I investigated the relationship between healthy and non-healthy reciepes. This first table shows the means of healthy and non healthy recipes for all quantitative columns, and the second is the median. I did both as median is not influence by outliers but it was interesting to see the differences. 

| is_healthy | rating | calories (#) | total fat (PDV) | sugar (PDV) | sodium (PDV) | protein (PDV) | saturated fat (PDV) | carbohydrates (PDV) |
| ---------: | -----: | -----------: | --------------: | ----------: | -----------: | ------------: | ------------------: | ------------------: |
| False      | 4.69   | 433.66       | 36.06           | 60.02       | 28.89        | 34.56         | 44.98               | 12.21               |
| True       | 4.65   | 348.18       | 10.99           | 83.14       | 31.12        | 26.18         | 11.84               | 18.50               |

| is_healthy | rating | calories (#) | total fat (PDV) | sugar (PDV) | sodium (PDV) | protein (PDV) | saturated fat (PDV) | carbohydrates (PDV) |
| ---------: | -----: | -----------: | --------------: | ----------: | -----------: | ------------: | ------------------: | ------------------: |
| False      | 5.0    | 317.1        | 23.0            | 20.0        | 16.0         | 20.0          | 27.0                | 8.0                 |
| True       | 5.0    | 238.1        | 7.0             | 32.0        | 11.0         | 13.0          | 6.0                 | 12.0                |

The biggest differences in the means between healthy and non-healthy were the calories and fats of each recipe. Interestingly, in the mean sugar for non-healthy is lower than for healthy recipes, which I would think would be the opposite. This trend stas the same in the medians, meaning that healthier recipes may tend to have lower fats overall.

## Assessment of Missingness

There are three columns, `'date'`, `'rating'`, and `'review'`, in the merged dataset have a significant amount of missing values, so I decided to assess the missingness on the dataframe.

### NMAR Analysis

I believe that the missingness of the `'review'` column is NMAR. These could be people who have interacted with the post, by giving a rating, but have not left a review simply because the recipe didn't leave a lasting impact on them. They also simply might have forgotten to leave a review. This is why `'review'` is NMAR, because people have either decided or forgotten to leave a review.

### Missingness Dependency

We moved on to examine the missingness of `'rating'` in the merged DataFrame by testing the dependency of its missingness. We are investigating whether the missiness in the `'rating'` column depends on the column `'n_ingredients'`, which is the number of ingredients in a recipe, or the column `'minutes'`, which is number of minutes need to complete the recipe.

> Proportion of Sugar and Rating

**Null Hypothesis:** The missingness of ratings does not depend on the number of ingredients in the recipe.

**Alternate Hypothesis:** The missingness of ratings does depend on the number of ingredients in the recipe.

**Test Statistic:** The absolute difference of mean in the number of ingredients of the distribution of the group without missing ratings and the distribution of the group without missing ratings.

**Significance Level:** 0.05

We ran a permutation test by shuffling the missingness of rating for 1000 times to collect 1000 simulating mean differences in the two distributions as described in the test statistic.

<iframe
  src="assets/empirical_diff_n_ingredients.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The **observed statistic** of **0.1607** is indicated by the red vertical line on the graph. Since the **p_value** that we found **(0.0)** is < 0.05 which is the significance level that we set, we **reject the null hypothesis**. The missingness of `'rating'` does depend on the `'n_ingredients'`, which is number of ingredients in the recipe.

> Minutes and Rating

**Null Hypothesis:** The missingness of ratings does not depend on the cooking time of the recipe in minutes.

**Alternate Hypothesis:** The missingness of ratings does depend on the cooking time of the recipe in minutes.

**Test Statistic:** The absolute difference of mean in cooking time of the recipe in minutes of the distribution of the group without missing ratings and the distribution of the group without missing ratings.

**Significance Level:** 0.05

<iframe
  src="assets/empirical_diff_prescale.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

We ran another permutation test by shuffling the missingness of rating for 1000 times to collect 1000 simulating mean differences in the two distributions as described in the test statistic.

<iframe
  src="assets/empirical_diff_minutes.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The **observed statistic** of **51.4524** is indicated by the red vertical line on the graph. Since the **p-value** that we found **(0.115)** is > 0.05 which is the significance level that we set, we **fail to reject the null hypothesis**. The missingness of rating does not depend on the cooking time in minutes of the recipe.

## Hypothesis Testing

As mentioned in the introduction, we are curious about whether people rate healthy recipes and non-healthy recipes on the same scale. By healthy recipes, I am talking about recipes that have the tag 'healthy' in their `'tags'` columns. 

To investigate the question, we ran a **permutation test** with the following hypotheses, test statistic, and significance level.

**Null Hypothesis:** People rate all the recipes on the same scale.

**Alternative Hypothesis:** People rate recipes that are generally healthier lower than unhealthy recipes.

**Test Statistic:** The difference of mean in the rating of healthy and non-healthy recipes.

**Significance Level:** 0.05

The reason we chose to run a permutation test is because we do not have any information of any population, and we want to check if the two distributions look like they come from the same population. I proposed that **people rate the healthier recipes lower** because people might think that healthier recipes taste worse because they need healthier ingredients. Since I would like to know all the opinions from the users, we used rating instead of average rating of the recipes. For the test statistic, I chose the difference in mean of the ratings of two groups of recipes instead of absolute difference in mean. This is because I have a directional hypothesis, which is that people rate healthier recipes lower than other recipes. By looking at the difference in mean between the two groups, we can see what type of recipes typically have a higher rating, which answers our question.

To run the test, we first split the data points into two groups: healthy, and the rest of the data points are in the non-healthy group. The **observed statistic** is **-0.0398**.

Then we shuffled the ratings for 1000 times to collect 1000 simulating mean differences in the two distributions as described in the test statistic. We got a **p-value** of **0.0**.

<iframe
  src="assets/empirical_diff_rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

#### Conclusion of Permutation Test

Since the **p-value** that we found **(0.0)** is less than the significance level of 0.05, we **reject the null hypothesis**. People do not rate all the recipes on the same scale, and they tend to rate healthier recipes lower. One plausible explanation for this founding could be that people dislike the alternative, healthier substitutions for recipes, making them rate them lower than their counterparts.

## Framing a Prediction Problem

I plan to predict the average number of calories in a recipe, which is a regression problem since I am predicting a continuous numerical value. To address my prediction problem, I will build a regression model to estimate the calorie content based on various features of the recipe.

I chose the average number of calories as my response variable because it is the most important factor in nutrition when trying to diet (caloric deficit vs caloric surplus). By understanding the caloric content, I can offer valuable nutritional information to users, which is essential for dietary planning and health management.

To evaluate my model, I will use metrics such as Mean Squared Error (MSE), Root Mean Squared Error (RMSE), and Mean Absolute Error (MAE). These metrics are suitable for regression problems and will provide a comprehensive evaluation of my model's performance. Given that the distribution of calorie counts may be skewed, these metrics will help me understand both the overall performance and the presence of any large errors.

The information I have prior to making my prediction includes all the columns in the recipe dataset, as listed in the introduction section. These columns represent features relating to the recipes themselves, such as ingredient types, quantities, preparation time, cooking methods, and other relevant characteristics. Since these features are intrinsic to the recipes, I can use them to predict the calorie content even if no one has made a specific calorie measurement or review on them.

## Baseline Model

For my baseline model, I am utilizing a regression model and splitting the data points into training and test sets. The features I am using for this model include 'total fat (PDV)', 'protein (PDV)', 'carbohydrates (PDV)', and 'is_healthy', which contain quantitative numerical values, and 'is_healthy', which contains nominal values since they are boolean values.

I one-hot encoded the boolean values in 'is_healthy' with the corresponding 0 and 1 values and dropped one of the encoded columns. This step allows me to train the model appropriately.

The metric, Root Mean Squared Error (RMSE), of this model is 38.75. The RMSE indicates the root average squared difference between the predicted calorie values and the actual calorie values in the test set. A lower RMSE represents a model that predicts closer to the actual values. In my case, the RMSE value suggests that there is room for improvement in the model's predictive accuracy. The distribution of calorie counts may be skewed, which could impact the model's performance.

By evaluating the RMSE, I aim to iteratively improve the model and achieve better predictive accuracy. The reason for focusing on these metrics is to ensure a comprehensive evaluation of the model's performance, particularly given the potential for skewed data distributions.

## Final Model

In the final model I added `n_ingredients`, `sugar(PDV)`, `sodium (PDV)`, `saturated fat (PDV)`, and `'n_ingredients'`. I believe that adding more ingredients into the recipe will increase the number of macronutrients, increasing the number of calories overall. The other nutrition facts help us identify other potential increases/decreases to calories as the PDV of each directly relates to the number of calories. For example, per gram of sugar there are 4 calories, which will help us predict the calories for each recipe better in the final model.

I used *k*-fold cross-validation to find the best hyperparameters for the regression model, which were the ones I decided to add. 

The metric, **RSME**, of the final model is *33.75**, which is a 5.0 increase from the RSME of the baseline model.

## Fairness Analysis

For the fairness analysis, I split the recipes into two groups: recipes submitted in and before 2010 and recipes submitted after 2010. I chose to evaluate the **RSME** of the model for the two groups, because that gives us the best idea of the accuracy of our regression model. 

**Null Hypothesis**: The model is fair. Its RSME for recipes posted after 2010 and before or in 2010 are roughly the same, and any differences are due to random chance.

**Alternate Hypothesis**: The model is unfair. Its RSME for recipes posted after 2010 is lower than its RSME for recipes with recipes posted in or before 2010.

**Test Statistic**: Difference in RSME (before or in 2010 - after 2010)

**Significance Level**: 0.05

<iframe
  src="assets/empirical_rsme.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

To run the permutation test, I binarized the `'submitted'` column to have 0 or 1 if the recipe was submitted in 2010 or before or after 2010 respectively. When I took the difference in the RSME, we got an observed test statistic of **9.49**. I then shuffled the `'submitted'` column 1000 times to collect 1000 simulated differences in the two distributions as described in the test statistics. After running the permutation test, I got a p-value of **0.36**. Since the p-value of 0.36 is greater than 0.05, we fail to reject the null hypothesis that our model is fair. The model's RSME for recipes submitted before and in 2010 are roughly the same as recipes posted after 2010.

---
