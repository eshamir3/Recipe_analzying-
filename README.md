# Introduction 

# Data Cleaning and Exploratory Data Analysis 
To clean our data, we started by preforming a left merge on our recipe and interaction datasets into one dataset containing all information about each recipe as well as each review given on each recipe. We chose to use a left merge in order to preserve recipes that reviews were not written about in our dataset. Next, we replaced any 0 values in 'rating' rating with np.nan. We did this because the dataset specifies that ratings are given on a scale of 1-5, so therefore, any values of 0 represent a missing rating. We added a column to dataset called 'avg_rating' to represent the average rating on each recipe. To do this, we grouped by each recipe, and extracted the average rating for each 'id'. Lastly, we converted both the date the recipe was created ('date'), and the date the review was submitted ('submitted') to datetime in case we wanted to preform further analysis on the dates. 

<iframe
  src="rating_distributions.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

# Asssessment of Missingness
## NMAR Analysis 
I believe that one column that has a non-trivial missingness and could be NMAR is the review column. The review column corresponds to the written review for a review/rating for a single recipe by a user. The review column is NMAR because people are more likely to leave a written review for recipes that they feel strongly about --either strong positive feels or strong negative feelings. Therefore, reviews or sentiment that is relatively neutral or ambivalent will be missing from the review column, showing that the the review column is NMAR. 

# Hypothesis Testing 

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
# Fairness Analysis
