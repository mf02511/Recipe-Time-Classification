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

## Final Model

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