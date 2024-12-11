# Overview 

This project focuses on analyzing the relationship between recipe attributes—such as the number of ingredients, steps, and user feedback—and recipe ratings. Additionally, it explores patterns in user reviews, investigates missing data dependencies, and develops a predictive model for estimating recipe cook times. Through these analyses, we aim to uncover trends that impact user engagement and satisfaction.

# Introduction 

Our analysis examines key factors that influence recipe ratings, such as the complexity of recipes, the number of ingredients, and the length of reviews. We hypothesize that simplicity, as reflected by fewer ingredients and shorter steps, may correlate with higher ratings due to its appeal to busy individuals. At the same time, review length may reveal user sentiment, with detailed feedback indicating mixed opinions or strong feelings. The question that we chose to analyze through this project was: "How do various factors impact the cook time of a recipe?" 

Beyond understanding cook time, we also investigate missing data patterns, exploring whether the absence of recipe descriptions or reviews depends on other variables, such as the number of ingredients or the length of user reviews. These insights help us address biases in the dataset and enhance the reliability of our findings.

Lastly, we develop a predictive model to categorize recipes into practical cook time bins, such as "Weekday Recipe" or "Holiday Recipe," based on factors like ingredients, steps, and nutritional data. This classification approach aims to provide users with a quick way to gauge how much time a recipe might require, making cooking more accessible and manageable.

Our project leverages two datasets from Food.com: one detailing recipe metadata and another containing user reviews and ratings. These datasets, originally compiled for research into personalized recipe recommendations, provide a foundation for our exploration recipe and review characteristics.

# Data Cleaning and Exploratory Data Analysis

To clean our data, we started by performing a left merge on our recipe and interaction datasets into one dataset containing all information about each recipe as well as each review given on each recipe. We chose to use a left merge in order to preserve recipes that reviews were not written about in our dataset. Our merged dataset initally contained the following columns:

| **Column Name**       | **Data Type**        |
|------------------------|----------------------|
| **`name`**            | `object`            |
| **`id`**              | `int64`             |
| **`minutes`**         | `int64`             |
| **`contributor_id`**  | `int64`             |
| **`submitted`**       | `datetime64[ns]`    |
| **`tags`**            | `object`            |
| **`nutrition`**       | `object`            |
| **`n_steps`**         | `int64`             |
| **`steps`**           | `object`            |
| **`description`**     | `object`            |
| **`ingredients`**     | `object`            |
| **`n_ingredients`**   | `int64`             |
| **`user_id`**         | `float64`           |
| **`date`**            | `datetime64[ns]`    |
| **`rating`**          | `float64`           |
| **`review`**          | `object`            |

Next, we replaced any `0` values in `rating` rating with `np.nan`. We did this because the dataset specifies that ratings are given on a scale of 1-5, so therefore, any values of `0` represent a missing rating. We added a column to the dataset called `avg_rating` to represent the average rating on each recipe. To do this, we grouped by each recipe, and extracted the average rating for each `id`. Lastly, we converted both the date the recipe was created `date`, and the date the review was submitted `submitted` to datetime in case we wanted to perform further analysis on the dates. 

Next, we created a column called is_dessert, which identifies whether a recipe is classified as a dessert. This was done by checking if the word "dessert" appears in the tags column. Missing values in the tags column were replaced with empty strings to prevent errors, and the results were converted to a binary format (1 for dessert and 0 otherwise).

We also cleaned the nutrition data to make it more structured and useful. The raw nutritional information, originally stored as strings with square brackets, was processed by removing the brackets, splitting the data into individual components, and converting these components into numeric values. This transformed the nutrition data into lists of floating-point numbers, where each element represents a specific nutritional metric.

From this cleaned nutritional data, we extracted two specific metrics: calories and sodium. The calorie content for each recipe was extracted as the first value in the cleaned nutrition list and stored in a new calories column. Similarly, the sodium content, represented as the fourth value in the list (in milligrams), was stored in a new sodium column. These steps made it easier to analyze recipes based on their dessert classification and nutritional properties. 

After performing the left merge detailed in the Data Cleaning section of the website, our DataFrame had 234429 rows and 16 columns.

| Column Name       | Description                                     |
|--------------------|-------------------------------------------------|
| `name`            | Name of the recipe.                            |
| `id`              | Unique identifier for the recipe.              |
| `minutes`         | Time required to make the recipe (in minutes). |
| `contributor_id`  | ID of the person who contributed the recipe.   |
| `submitted`       | Date the recipe was submitted.                 |
| `tags`            | Tags associated with the recipe.              |
| `nutrition`       | Nutritional information, cleaned and formatted.|
| `n_steps`         | Number of steps in the recipe.                |
| `steps`           | Detailed steps for the recipe.                |
| `description`     | Description of the recipe.                    |
| `ingredients`     | List of ingredients.                          |
| `n_ingredients`   | Number of ingredients.                        |
| `user_id`         | ID of the user who interacted with the recipe.|
| `date`            | Date of the interaction.                      |
| `rating`          | User-provided rating for the recipe.          |
| `review`          | User-provided review for the recipe.          |
| `avg_rating`      | Average rating for the recipe.                |
| `len_review`      | Length of the review text.                    |
| `is_dessert`      | Indicates whether the recipe is for a dessert (1 for dessert, 0 otherwise). |
| `calories`        | Estimated calories for the recipe.            |
| `sodium`          | Estimated sodium content (in milligrams) for the recipe. |


The head of our cleaned dataframe is: 



| **`name`**                | **`id`**    | **`minutes`** | **`contributor_id`** | **`submitted`** | **`tags`**              | **`nutrition`**          | **`n_steps`** | **`description`**    | **`n_ingredients`** | **`user_id`** | **`date`**    | **`rating`** | **`review`**          | **`avg_rating`** | **`len_review`** | **`is_dessert`** | **`calories`** | **`sodium`** |
|----------------------------|-------------|---------------|----------------------|-----------------|-------------------------|--------------------------|---------------|----------------------|---------------------|---------------|---------------|-------------|----------------------|------------------|------------------|------------------|----------------|--------------|
| 1 brownies in the wo...    | 333281      | 40            | 985201               | 2008-10-27      | [‘60-minutes-or-le...   | [138.4, 10.0, 50.0...]   | 10            | These are the mos... | 9                   | 386585        | 2008-11-19    | 4           | These were prett...  | 4                | 254              | 1                | 138.4          | 3            |
| 1 in canada chocolat...    | 453467      | 45            | 1848091              | 2011-04-11      | [‘60-minutes-or-le...   | [595.1, 46.0, 211...]   | 12            | This is the rec...   | 11                  | 424680        | 2012-01-26    | 5           | Originally I was...  | 5                | 336              | 0                | 595.1          | 22           |
| 412 broccoli cassero...    | 306168      | 40            | 50969                | 2008-05-30      | [‘60-minutes-or-le...   | [194.8, 20.0, 6.0...]   | 6             | Since there are ...  | 9                   | 29782         | 2008-12-31    | 5           | This was one of ...  | 5                | 469              | 0                | 194.8          | 32           |
| 412 broccoli cassero...    | 306168      | 40            | 50969                | 2008-05-30      | [‘60-minutes-or-le...   | [194.8, 20.0, 6.0...]   | 6             | since there are ...  | 9                   | 768828        | 2013-08-02    | 5           | Loved this. Be s...  | 5                | 188              | 0                | 194.8          | 32           |
| 412 broccoli cassero...    | 306168      | 40            | 50969                | 2008-05-30      | [‘60-minutes-or-le...   | [194.8, 20.0, 6.0...]   | 6             | I made this for ...  | 9                   | 1196280       | 2009-04-13    | 5           | I made this for ...  | 5                | 162              | 0                | 194.8          | 32           |



## Univariate Analysis: 
In our exploratory data analysis, we first perform univariate analysis to examine the distribution ratings across the entire dataframe.
<iframe
  src="rating_distributions.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
According to our univariate analysis, the majority of reviews contain a rating of 5 stars, with over 90% of ratings being given between 3-5 stars. Reviews rarely contained ratings of one and two. 

## Bivariate Analysis:

<iframe
  src="minutes_vs_steps.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
This plot shows the distribution of average cook time for each unique number of steps in the dataset. As we can see, there is a sharp increase in the cook time as the number of steps increases. This relationship may not be linear, but it does appear that there is a positive relationship between average number of minutes (average cooktime) and average number of steps. 

# Interesting Aggregates

## Aggregate analysis of average number of ingredients per rating:
 
The Aggregation of "Average Number of Ingredients by Rating" provides valuable insights into the relationship between recipe ratings and the average number of ingredients. This analysis can help identify trends in user preferences—whether higher-rated recipes tend to be simpler (fewer ingredients) or more elaborate (greater number of ingredients). For instance, if recipes with fewer ingredients receive higher ratings, it could indicate a preference for convenience. In our grouped analysis, we found that the average number of ingredients per rating is nearly uniform. 

# Average Number of Ingredients Per Rating

| Rating | Average Number of Ingredients |
|--------|-------------------------------|
| 1      | 2.0                           |
| 2      | 4.0                           |
| 3      | 5.5                           |
| 4      | 7.0                           |
| 5      | 9.0                           |


## Aggregate analysis of number of recipes per contributor:

This aggregation the number of recipes contributed by the top 10 contributors, showcasing their significant role in the dataset. The contributor with ID `37449` stands out, having submitted over 3,000 recipes, making them the highest contributor by a notable margin. Other contributors, such as `226863` and `424680`, follow closely. This part of our analysis demonstrates that many of our recipes come from the same contributor, which could explain certain patterns in our data. 

# Average Number of Recipes Per Contributor 

| Contributor ID  | Average Number of Recipes |
|---------|-------------------------------|
| 37449   | 3060                          |
| 226863  | 2754                          |
| 424680  | 2503                          |
| ...     | ...                           |
| 65502   | 1795                          |
| 679953  | 1711                          |
| 461834  | 1680                          |



## Aggregate analysis of Average Review Length per Rating:

This agreggate plot, "Average Review Length by Rating," highlights the relationship between the average length of user reviews and their assigned recipe ratings. The trend suggests that users who rate recipes with a score of 3 tend to write the longest reviews, averaging over 340 characters. This may indicate that users with mixed feelings (neither entirely positive nor negative) take more time to explain their experiences or provide detailed feedback.
Conversely, the shortest reviews are associated with extreme ratings of 1 and 5. These users might be more inclined to provide concise feedback, such as quick praise or criticism. This pattern provides insights into user engagement and how satisfaction levels influence the length of reviews, which can be valuable for understanding user sentiment and feedback quality.

| Rating | Average Review Length by Rating |
|--------|---------------------------------|
| 1.0    | 250.57                         |
| 2.0    | 298.07                         |
| 3.0    | 335.85                         |
| 4.0    | 314.88                         |
| 5.0    | 287.15                         |




# Asssessment of Missingness
## NMAR Analysis 
I believe that one column that has a non-trivial missingness and could be NMAR is the review column. The review column corresponds to the written review for a review/rating for a single recipe by a user. The review column is NMAR because people are more likely to leave a written review for recipes that they feel strongly about --either strong positive feels or strong negative feelings. Therefore, reviews or sentiment that is relatively neutral or ambivalent will be missing from the review column, showing that the the review column is NMAR. 

## Missingness Dependency of Description on Number of Ingredients 
We preformed a permutation test in order to determine wether the missingness of a description was MAR dependent on any other columns. First, we examined the mean number of ingredients when Description is missing versus not missing. 

**Null Hypothesis**: The mean number of ingredients is the same when Description is missing versus not missing.

**Alternate Hypothesis**:  The mean number of ingredients is different when Description is missing versus not missing.

<iframe
  src="missing_MAR.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

We conducted a permutation test, performing 1,000 simulations to generate an empirical distribution of the test statistic under the null hypothesis. This approach allowed us to assess the likelihood of observing the calculated test statistic, or one more extreme, under the assumption that the null hypothesis is true. 

Our analysis yielded a p-value of 0.005, which falls below our chosen significance level of 0.05. This indicates strong evidence against the null hypothesis and leads us to reject it with confidence. Based on these findings, we conclude that there is a statistically significant difference in the mean number of ingredients between instances where the Description is missing and where it is not. This result suggests that the presence or absence of a Description is associated with variations in the mean number of ingredients.



## Missingness Dependency of Description on Length of Review 
Next, we aimed to find a column on of with missingness of Description was not dependent on. We chose to analyze the relationship between missingness of the description column and Length of review.

**Null Hypothesis**: The mean length of review is the same when Description is missing versus not missing.

**Alternate Hypothesis**:  The mean length of review is different when Description is missing versus not missing.

<iframe
  src="not_missing_MAR.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

We conducted a permutation test with 1,000 simulations to assess whether the length of the review is associated with the missingness of the Duration column. This analysis aimed to evaluate the relationship under the null hypothesis, which assumes no dependence between review length and the missingness of the Duration column. The resulting p-value was 0.888, which is considerably higher than our chosen significance level of 0.05.

Since our p-value is significantly higher than our significance level, we fail to reject the null hypothesis. This lack of statistical significance suggests that there is no evidence to support a relationship between review length and whether the Duration column is missing. Therefore, we accept the null hypothesis that the missingness of the Duration column is not related to the average length of the reviews.



# Hypothesis Testing 
For our hypothesis test we aimed to determine if the rating has an effect on the number of steps in the recipes. This will be achieved using a one-way ANOVA test.

**Null Hypothesis**: The rating of each recipe does not have a relationship to mean number of steps.

**Alternate Hypothesis**:  The rating of each recipe does have a relationship to mean number of steps, with mean number of steps differing according to the rating.

The dataset was first cleaned to remove any rows with missing values in the `rating` column and the `n_steps' column. The data was then grouped by rating (from 1 to 5), and the corresponding number of steps was extracted in order to preform the one-way  ANOVA test. The one-way ANOVA test yielded an F-statistic of F= 49.83, with a p-value of 0. Since the p-value is significantly smaller than our chosen threshold of 0.05, we find strong evidence to reject the null hypothesis. 

A one way ANOVA is appropriate in this case because we are aiming to discern whether there is a signficant difference between the distributions of multiple groups. An ANOVA test will tell us whether the variance in the data is more explained by the variance between groups is or better explained by the variance within groups. A small p-value from the ANOVA test indicates that the variance in the data is more explained by the variance between groups, indicating a significant difference between at least two of the groups, which fits the conditions under which we would reject the null hypothesis. Our p-value of 0 indicates that the rating does have a statistically significant effect on the number of steps in recipes. These findings suggest that the average number of steps varies meaningfully across recipes with different ratings. This also means, that number of steps can be used as a predictory of 'rating'.


# Framing a Prediction Problem
Our predictive model aimed to predict the cook time of a recipe. 

 We created a new column in the dataset, called "minute_category" that transformed the minutes column from a numerical discrete variable to a nomial variable. 
The categories that the cook time was divided into are: 
- Weekday Recipe (less than an hour of cook time)
- Weekend Recipe (1-3 hours of cook time)
- Holiday/Special Occasion Recipe (3-6 hours of cook time)
- Long Term Recipe (more than 6 hours of cook time)

We envsion this predictive model as a way for people to gauge a general amount of time that it would take for them to cook. Many indviduals would consider a recipe that takes 1 hour and a recipe that takes 1 hour and 10 minutes to take the essentially the same amount of time. A regression model would treat these two cooktimes as being different, and would penalize us for a predction that is not exactly the number of minutes of a recipe, which does not reflect the feelings of many individuals who cook. A classifier is better suited for our predictive problem, as it allows us to place the cook time into bins that better reflect how individuals think of different categories of cook time of recipes in their head. 

We are performing multi-class classification. We used accuracy as our test static, as accuracy weighs false negatives and false positives the same. Because this is prediction of cooktime, false negatives and false positives are equally detrimental, one is not more preferred or more penalized than the other. Because of the equal weighing of false negatives and false positives, we chose accuracy as our metric. 

At the time of prediction, we envision that someone would have all the informaion displayed on the general page for a recipe, minus the number of minutes the recipe would take. They would have access to the number of steps and the descriptions of those steps, the numer of ingredients, the average rating of the recipe, and the nutrition information. They would also be able to tell at a glance whether the recipe was for a dessert or not. We will use only these features to predict the minutes category of a recipe. 

After dropping features that are not helpful for our analysis, namely description and steps (but not number of steps), we are left with the dataframe we will use for our model. 

The head of the dataframe that we wil be using for our model is:

|   minutes |   n_steps |   n_ingredients |   avg_rating |   is_dessert |   calories |   sodium | minutes_category   |
|----------:|----------:|----------------:|-------------:|-------------:|-----------:|---------:|:-------------------|
|        40 |        10 |               9 |            4 |            1 |      138.4 |        3 | Weekday            |
|        45 |        12 |              11 |            5 |            0 |      595.1 |       22 | Weekday            |
|        40 |         6 |               9 |            5 |            0 |      194.8 |       32 | Weekday            |
|        40 |         6 |               9 |            5 |            0 |      194.8 |       32 | Weekday            |
|        40 |         6 |               9 |            5 |            0 |      194.8 |       32 | Weekday            |


# Baseline Model

Our baseline model was a KNN classifier that used two quantitative columns, 'n_ingredients' and 'n_steps' to predict the category of cooktimes (the 'minute_category'). The 'minutes category' column is a column of categorical nominal data. We used only numerical data for the features of this model, so we had no need to peform any encodings, whether that be ordinal encodings or One Hot Encodings. 

Intitally, the accuracy of our model was 0.70, or 70% on the training and test data. This indicates that the model was consistent in its accuracy between test and training data, indcating that it was a good model. Still, there were ways to optimize the model to make it better.  

# Final Model

In our final model, we incorporated the following features: n_ingredients, n_steps, calories, sodium, and is_dessert. We used a KNeighborsClassifier and were able to achieve an accuracy score of 0.83 on the test set. The KNN classifer works by estimating where a given set of features would fit in the space of its training data, and takes the labels of the n neighbors closest to it, and the label that is most frequent in the n neighbors is returned as the predicted label for the data point. 

We first standardized the features `calories` and `sodium`  using StandardScaler, as they are on different scales and could skew the performance of the model. Additionally, calories and sodium having a value of 0 has no real world meaning, as it is very unlikely that a recipe would have 0 calories of 0% of the PDV of sodium, so it makes more sense to standardize those columns so that we can determine the relative amounts of sodium and calories of each recipe. The features `n_ingredients` and `n_steps` were included as they are, while `is_dessert` was one-hot encoded to account for its categorical nature, as described in step 1.

`calories` and `sodium` were included becasue we hypothesized that foods with relatively higher sodium would have shorter cook times. For example, instant ramen is very widely known to be a cost effective and quick dinner, and it takes only a couple minutes to prepare. In general, processed or ultra-processed food contain higher levels of sodium, and many reach for processed or ultra-processed foods as a way of speeding up cook time (so one does not have to make everything from scratch). Additionally, we theorized that foods that contained relatively lower calories would take a shorter time to cook. For example, a snack takes less time to make than a dinner, and snacks are usually less calorically dense than a full meal. From the perspective of the data generating process, we hypothesized that because there seemed to be a relationship between these columns and the trend of minutes, they would be more informative features, and would allow our model to make better predictions. 

We used GridSearchCV to find the best hyperparameters for the KNeighborsClassifier. The hyperparameters we tuned were `n_neighbors`, `weights`, and `metric`. The optimized values were:

`n_neighbors`: `5`
`weights`: `uniform`
`metric`: `manhattan`

After tuning the hyperparameters, the best model achieved an improved accuracy score of 0.91 on the training set. The use of cross-validation ensured the model's robustness. Since the accuracy increased from the baseline to the final model, this indicates better performance of the optimized KNeighborsClassifier.


# Fairness Analysis
We performed a fairness analysis to determine whether there was accuracy parity between recipes that had a high average rating and a low average rating. We set the threshold for high ratings to be above 3, so any recipe that was rated over 3 was considered to be highly rated, while any recipe that had an average rating of below 3 was considered to not be highly rated. We created these groups by using a helper function to binarize the “average rating” column. 

Our **null hypothesis** was that there is no difference in accuracy between low and high rated recipes. 

Our **alternative hypothesis** was that there is a difference in accuracy between low and high rated recipes. 

With similar reasoning as for why we selected accuracy as our metric for our classifier, we chose accuray parity for our fairness analysis because we wish to weigh false negatives and false positives for each category equally, and precision and recall do not weigh them equally. We chose a significance level of 0.05 and performed a permutation test with 500 repetitions to simulate the distribution under the null hypothesis. 

Our p-value for our permutation test was 1.0, which is higher than our significance level of 0.05, so we fail to reject the null hypothesis that there is no difference in the classification accuracy between low and high rated recipes, and that our model has achieved accuracy parity for low and high rated recipes. 

