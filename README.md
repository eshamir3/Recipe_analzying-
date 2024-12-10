# Introduction 

# Data Cleaning 
To clean our data, we started by preforming a left merge on our recipe and interaction datasets into one dataset containing all information about each recipe as well as each review given on each recipe. We chose to use a left merge in order to preserve recipes that reviews were not written about in our dataset. Next, we replaced any `0` values in `rating` rating with `np.nan`. We did this because the dataset specifies that ratings are given on a scale of 1-5, so therefore, any values of `0` represent a missing rating. We added a column to dataset called `avg_rating` to represent the average rating on each recipe. To do this, we grouped by each recipe, and extracted the average rating for each 'id'. Lastly, we converted both the date the recipe was created `date`, and the date the review was submitted `submitted` to datetime in case we wanted to preform further analysis on the dates. 



# Exploratory Data Analysis 
Univariate Analysis : In our exploratory data analysis, we first perform univariate analysis to examine the distribution ratings across the entire dataframe.

<iframe
  src="rating_distributions.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

According to our univariate analysis, the majority of reviews contain a rating of 5 stars, with over 90% of ratings being given between 3-5 stars. Reviews rarely contained ratings of one and two. 

Bivariate Analysis: Next, in our exploratory data analysis, we preformed a bivariate analysis in order to examine the relationship between rating and number of steps.

## INSERT PLOT 

According to our bivarariate analysis, there exists a relationship between number of steps and rating. 

# Asssessment of Missingness
## NMAR Analysis 
I believe that one column that has a non-trivial missingness and could be NMAR is the review column. The review column corresponds to the written review for a review/rating for a single recipe by a user. The review column is NMAR because people are more likely to leave a written review for recipes that they feel strongly about --either strong positive feels or strong negative feelings. Therefore, reviews or sentiment that is relatively neutral or ambivalent will be missing from the review column, showing that the the review column is NMAR. 

## Missingness Dependency of Description on Number of Ingredients 
We preformed a permutation test in order to determine wether the missingness of a description was MAR dependent on any other columns. First, we examined the mean number of ingredients when Description is missing versus not missing. 

**Null Hypothesis**: The mean number of ingredients is the same when Description is missing versus not missing.

**Alternate Hypothesis**:  The mean number of ingredients is different when Description is missing versus not missing.

## INSERT PLOT OF MEANS AND WHERE OUR MEAN IS

We conducted a permutation test, performing 1,000 simulations to generate an empirical distribution of the test statistic under the null hypothesis. This approach allowed us to assess the likelihood of observing the calculated test statistic, or one more extreme, under the assumption that the null hypothesis is true. 

Our analysis yielded a p-value of 0.005, which falls below our chosen significance level of 0.05. This indicates strong evidence against the null hypothesis and leads us to reject it with confidence. Based on these findings, we conclude that there is a statistically significant difference in the mean number of ingredients between instances where the Description is missing and where it is not. This result suggests that the presence or absence of a Description is associated with variations in the mean number of ingredients.



## Missingness Dependency of Description on Length of Review 
Next, we aimed to find a column on of with missingness of Description was not dependent on. We chose to analyze the relationship between missingness of the description column and Length of review.

**Null Hypothesis**: The mean length of review is the same when Description is missing versus not missing.

**Alternate Hypothesis**:  The mean length of review is different when Description is missing versus not missing.

## INSERT PLOT OF MEANS AND WHERE OUR MEAN IS

We conducted a permutation test with 1,000 simulations to assess whether the length of the review is associated with the missingness of the Duration column. This analysis aimed to evaluate the relationship under the null hypothesis, which assumes no dependence between review length and the missingness of the Duration column. The resulting p-value was 0.888, which is considerably higher than our chosen significance level of 0.05.

Since our p-value is significantly higher than our significance level, we fail to reject the null hypothesis. This lack of statistical significance suggests that there is no evidence to support a relationship between review length and whether the Duration column is missing. Therefore, we accept the null hypothesis that the missingness of the Duration column is not related to the average length of the reviews.



# Hypothesis Testing 
For our hypothesis test we aimed to determine if the rating has an effect on the number of steps in the recipes. This will be achieved using a one-way ANOVA test.

**Null Hypothesis**: The rating of each recipe does not have a relationship to mean number of steps.

**Alternate Hypothesis**:  The rating of each recipe does have a relationship to mean number of steps, with mean number of steps differing according to the rating.

## INSERT PLOT OF MEANS FOR ANOVA

The dataset was first cleaned to remove any rows with missing values in the `rating` column and the `n_steps' column. The data was then grouped by rating (from 1 to 5), and the corresponding number of steps was extracted in order to preform the one-way  ANOVA test. The one-way ANOVA test yielded an F-statistic of F= 49.83, with a p-value of 0. Since the p-value is significantly smaller than our chosen threshold of 0.05, we find strong evidence to reject the null hypothesis. This result indicates that the rating does have a statistically significant effect on the number of steps in recipes. These findings suggest that the average number of steps varies meaningfully across recipes with different ratings. This also means, that number of steps can be used as a predictory of 'rating'.


# Framing a Prediction Problem
Our predictive model aimed to predict the cook time of a recipe. 

Initially, we aimed to use a regression model to predict the cook time (in minutes of a recipe), but we noticed that the high amount of outliers in terms of cook time was negatively affecting our model. Our baseline linear regression model had a negative value for RMSE, which indicates that our model was performing worse than a constant model that would just predict the mean. We observed the same behavior even after removing outliers using IQR analysis. 

For these reasons, we decided to switch our prediction problem from regression to classification. We created a new column in the dataset, called "minute_category" that transformed the minutes column from a numerical continuous variable to a nomial variable. 
The categories that the cook time was divided into are: 
- Weekday Recipe (less than an hour of cook time)
- Weekend Recipe (1-3 hours of cook time)
- Holiday/Special Occasion Recipe (3-6 hours of cook time)
- Long Term Recipe (more than 6 hours of cook time)

We envsion this predictive model as a way for people to gauge a general amount of time that it would take for them to cook. Many indviduals would consider a recipe that takes 1 hour and a recipe that takes 1 hour and 10 minutes to take the essentially the same amount of time. Our regression model would treat these two cooktimes as being different, and would penalize us for a predction that is not exactly the number of minutes of a recipe, which does not reflect the feelings of many individuals who cook. This is another reason why a classifier is better suited for our predictive problem, as it allows us to place the cook time into bins that better reflect how individuals view the cook time of recipes in their head. 

We are performing multi-class classification. Because we are not performing binary classification, we used accuracy the metric used to evaulate our model, as we cannot use precision, recall, or the F-1 statistic. 

# Baseline Model

Our baseline model was a KNN classifier that used two quantitative columns, 'n_ingredients' and 'n_steps' to predict the category of cooktimes (the 'minute_category'). The 'minutes category' column is a column of categorical nominal data. 

Intitally, the accuracy of our model was 0.70, or 70%. 

# Final Model

In our final model, we incorporated the following features: n_ingredients, n_steps, calories, sodium, and is_dessert. We used a KNeighborsClassifier and were able to achieve an accuracy score of 0.83 on the test set. We first standardized the features `calories` and `sodium`  using StandardScaler, as they are on different scales and could skew the performance of the model. The features `n_ingredients` and `n_steps` were included as they are, while `is_dessert` was one-hot encoded to account for its categorical nature.

We used GridSearchCV to find the best hyperparameters for the KNeighborsClassifier. The hyperparameters We optimized were:

`n_neighbors`: `5`
`weights`: `uniform`
`metric`: `manhattan`

After tuning the hyperparameters, the best model achieved an improved accuracy score of 0.91 on the training set. The use of cross-validation ensured the model's robustness. Since the accuracy increased from the baseline to the final model, this indicates better performance of the optimized KNeighborsClassifier.


# Fairness Analysis
