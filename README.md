# Recipe Time Classification 🥘

**Authors**: Ashley Ho & Mizuho Fukuda
<br/>

**Course**: DSC80 at UCSD
<br/>

**Website**: <https://mf02511.github.io/Recipe-Time-Classification/>
<br/>

**GitHub**: <https://github.com/mf02511/Recipe-Time-Classification>
<br/>

**EDA**: <https://a1ho.github.io/Recipe-Ratings-Analysis/>

## Problem Identification
In this study, we explore the following prediction problem: 
> Predict whether or not a recipe is quick using a binary classification model.
>

For the purposes of this model, we define a recipe to be quick if it takes 30 minutes or less to prepare. In the exploratory data analysis we previously performed, we found that the mode amount of `'minutes'` for recipes in the `recipes` dataset was 30 minutes. Thus we chose 30 minutes as the threshold for our binary classification to produce an roughly even distribution of quick and non-quick recipes. 
<br/>

We perform some additional cleaning to the `recipes` dataset in preparation for our model creation. Prior to choosing the exact features we use in our model, we decided to drop all rows containing any missing values to ensure the model has access to complete information to make more accurate predictions. The `recipes` dataset originally had 83,782 different observations and after dropping rows with null values, the dataset has 72,251 different obesrvations. In addition, for each of the text features `'tags'`, `'steps'`, and `'ingredients'` in `recipes`, we combined the list of string elements for each observation into a single string. For the `'review'` column in the `interactions` dataset, we grouped by each recipe and aggregated all the reviews into a single string, which we then added to the `recipes` dataset.
<br/>

Now, for our prediction model, we have the following characteristics:
#### **Response Variable**
The response variable is `'minutes'`. As mentioned above, we are interested in the quickness of recipes, and we binarize the `'minutes'` column into 0 and 1 values, where 0 indicates non-quick recipes and 1 indicates quick recipes. We choose this to be our response variable because the amount of time a recipe takes to make important to people when choosing which recipes to try. After a long day, we often want to find a quick and easy recipe to make, and thus the quickness of a recipe may be crucial to an individual's decision making. Before binarizing into 0/1 categories, we dropped all rows with outlier values in the `'minutes'` column in order to make sure that all our of the values used in our model are resonable. We use the standard <(Q1 - 1.5(IQR)) and >(Q3 + 1.5(IQR)) formula to remove outliers. We do this specifically because there are several extreme outliers in the `'minutes` column that result from certain fake recipes. For example, there is a recipe with more than 1 million minutes of cooking time titled _'how to preserve a husband'_ , which is not meant to be taken seriously. We also note that from our data expoloration, there are no extremely small outliers in the `'minutes'` column.
<br/>

#### **Evaluation Metric**
The main metric we use to evaluate our model performance is precision. We note that in addition to precision, we also calculated accuracy, recall, and the F1-score on our models, but precision was the main metric we focus on for evalution. We choose precision because false positives are more harmful to our model. Suppose someone has had a really long day and just wants to make a quick meal -- if the model makes a lot of false positive classifications, this person would be at risk of choosing a recipe that is supposedly quick, but turns out to not actually be quick. Thus, it would be worse to falsely categorize a recipe as quick when it is not actually than falsely categorizing a recipe as not quick when it actually is. In the second situation, those recipes would simply be ignored and not be chosen by someone who wants to make a quick meal.
<br/>

#### **Known at Time of Prediction**
The `'minutes'` column is provided by the person who uploads the recipe. Thus, a user can view all the other information for that recipe on the website in order to predict the quickness of a recipe if it so happens that the `'minutes'` column is missing, or the person who uploads the recipe chooses not to upload the `'minutes'`. If the recipe has not been tried and rated/reivewed at least once before, then a user would not have access to the `'mean_rating'`, and `'review'` (which we grouped into a single string and added to the `recipes` dataset, as mentioned above) information. Otherwise, generally all other information in the `recipes` dataset will be known at the time of prediction and `'mean_rating'`, and `'review'` information will also be known if at least one other user has rated and reviewed the recipe before. 

## Baseline Model
We chose the Random Forest Classifier as the model for this classification task. The model includes three quantitative features: `n_ingredients`, `mean_rating`, and `calories`. In the pipeline, we first transform all features columns with using the Standard Scaler transformer. Since we do not have any categorical features for this baseline model, this quantitative scaling is the only transformation that was necessary in this case. We felt the Standard Scaler transformation was most appropriate since the feature columns have drastically different scales i.e. the `calories` column contains values in the hundreds while `n_ingredients` and `mean_rating` contain much smaller values. The difference in scale can lead to the `calories` feature having significantly larger weight than others, thus, transforming the features using Standard Scaler would produce a better model. The baseline Random Forest Classifier used default hyperparameters, namely, `max_depth=None`, `min_samples_split=2`, and `n_estimators=100`.


The following confusion matrix displays the baseline model's prediction on the test set:

![baseline confusion matrix](/assets/cm_base.png)

Performance Metrics:

|               |   Training |   **Testing** |
|:--------------|-----------:|--------------:|
| accuracy      |   0.959802 |      0.58318  |
| **precision** |   0.955937 |  **0.582119** |
| recall        |   0.964881 |      0.589907 |
| f1-score      |   0.960388 |      0.585987 |

We see that the baseline model has a very high training performance with a accuracy of 95.98% and a precisino score of 95.59%. However, the model performs poorly for unseen data (the test set) with an accuracy rate of 58.32% and precision score of 58.21%. This is expected as the baseline Random Forest Classifier used `max_depth=None`, which causes the classifier to overfit the training set significantly, leading to poor generalization. In general, we know that Decision Trees and Random Forests are very prone to overfitting if the depth of the tree is not restricted. Since we wish for the model to be able to predict on unseen data, this baseline model is very bad since the testing errors are extremely high. 


## Final Model
For the final model, we included the features `n_reviews` and `reviews` in addition to the features from the baseline model. The `n_reviews` feature can possibly provide information on the popularity of the recipe since the number of reviews is likely correlated with the number of people who uses the recipe. We used a Function Transformer that takes in the `reviews` column and creates a binary feature that indicates whether the reviews for each recipe contain certain key words. These key words include: 'quick', 'easy', 'fast', and 'simple'. The `reviews` feature was encoded with 1 if the reviews for the recipe contains any of the key words, and 0 if they do not. We believe these key words provide important information for whether the recipe was quick or not. In addition, we used both the Standard Scaler and Quantile Transfomers to scale the quantitative columns. In the baseline model, we used only Standard Sclaer transformation but after observing the distributions of each feature closely, we found that `calories` and `n_reviews` have extremely right-skewed distributions (as shown in the histograms below). In this case, it is better use a Quantile Transformer in order to reduce the skew in the data. `n_ingredients` and `mean_rating` are less obviously skewed, so we used the Standard Scaler transformation. 

<iframe src="assets/calories.html" width=800 height=600 frameBorder=0></iframe>

<iframe src="assets/n_reviews.html" width=800 height=600 frameBorder=0></iframe>

*Note: We did not have to remove any features since the Random Forest Classifier is not affected by irrelevant features*

### Grid Search Cross Validation for Hyperparameters
We performed a 5-fold Grid Search Cross Validation in order to find the best hyperparameters for the Random Forest Classifier. We focused on the `max_depth` and `min_samples_split` hyperparameters for our search since these can affect the model's ability to generalize to unseen data. We decided to use the default `n_estimators=100` since we found after a grid search (not included in final notebook) that the optimal `n_estimators` is around 100 regardless. The Grid Search CV found the optimal hyperparameters to be `max_depth=110` and `min_samples_split=0.05`. Notice that min_samples_split is a proportion, rather than the number of samples since we want the hyperparameter to also be able to generalize to any size data.

With the newly fitted model, we get a confusion matrix as shown below for the test set:

![baseline confusion matrix](/assets/cm_final.png)

Performance Metrics:

|               |   Training |   **Testing** |
|:--------------|-----------:|--------------:|
| accuracy      |   0.688127 |      0.688872 |
| **precision** |   0.695341 |  **0.692997** |
| recall        |   0.680722 |      0.678291 |
| f1-score      |   0.687954 |      0.685565 |

The final model has an improved testing performance with a accuracy rate of 68.89% and a precision score of 69.30%. Though the training accuracy is lower due to decreased tree depth and an increased minimum samples split, the model has an increased generalizability. It is also worth noting that the final model has an especially high precision score, which is important in the context of predicting minutes (cooking time) since the model should be able to reliably predict the truly qucik recipes so that the user does not have to cook for longer than expected. 

## Fairness Analysis
To perform a fairness analysis on our Final Model, we ask the following question:
> Does our Final Model perform better for recipes that have a mean rating of 5 (high rating) than it does for recipes that have a mean rating lower than 5 (low rating)?
>

We choose the `'mean_rating'` column to anaylze our model's fairness because we saw in our exploratory data analysis that the distribution of values in the `'mean_rating'` column is left-skewed, with a large proportion of recipies in the dataset have `'mean_rating'` of 5. Our evaluation metric is once again precision. In order to analyze fairness of our model, we run a permutation test with difference in precision as our test statistic. We choose a significance level of 0.05 and have the following hypotheses:
- **Null Hypothesis**: Our model is fair. The model's precision for high rated recipies and low rated recipes are equal. (i.e. precision(high rated recipes) - precsion(low rated recipes) = 0).
- **Alternate Hypothesis**: Our model is unfair. The model's precision for high rated recipes is higher than for low rated recipes (i.e. precision(high rated recipes) - precsion(low rated recipes) > 0)
- **Test Statistic**: model's precision for high rated recipes - model's precision for low rated recipes
<br/>

Here is the plot of the results from our permutation test using 100 permutations:
<iframe src="assets/fairness.html" width=800 height=600 frameBorder=0></iframe>

We get a p-value of 0.09, which is higher than the significance level of 0.05, and therefore **we fail to reject the null hypothesis**. This suggests that our model is fair, which is consistnent with our intuition. The results suggest that the value of `'mean_rating'` is not dependent on other features in the model and vice versa. Thus, the model's performance is likely not affected by the value of `'mean_rating'`.