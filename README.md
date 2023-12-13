# Recipe Time Classification 🥘

**Authors**: Ashley Ho & Mizuho Fukuda
<br/>

**Course**: DSC80 at UCSD
<br/>

**Website**: <https://mf02511.github.io/Recipe-Ratings-Model/>
<br/>

**GitHub**: <https://github.com/mf02511/Recipe-Ratings-Model>

*Our EDA for this dataset can be found [here](https://a1ho.github.io/Recipe-Ratings-Analysis/)*

## Problem Identification
After a long day at work or school, we often wish to make a quick and easy meal becuase we're very tired!!!!

In this study, we explore the following prediction problem: 
> Predict whether or not a recipe is quick using a binary classification model.
>

For the purposes of this model, we define a recipe to be quick if it takes 30 minutes or less to prepare. In the exploratory data analysis we previously performed, we found that the mode amount of `'minutes'` for recipes in the `recipes` dataset was 30 minutes. Thus we chose 30 minutes as the threshold for our binary classification to produce an roughly even distribution of quick and non-quick recipes. 
<br/>

We perform some additional cleaning to the `recipes` dataset in preparation for our model creation. Prior to choosing the exact features we use in our model, we decided to drop all rows containing any missing values to ensure the model has access to complete information to make more accurate predictions. The `recipes` dataset originally had 83,782 different observations and after dropping rows with null values, the dataset has 72,251 different obesrvations. In addition, for each of the text features `'tags'`, `'steps'`, `'ingredients'`, and `'reviews'` in `recipes`, we combined the list of string elements for each observation into a single string.
<br/>

Now, for our prediction model, we have the following characteristics:
### Response Variable
The response variable is `'minutes'`. As mentioned above, we are interested in the quickness of recipes, and we binarize the `'minutes'` column into 0 and 1 values, where 0 indicates non-quick recipes and 1 indicates quick recipes. We choose this to be our response variable because the amount of time a recipe takes to make important to people when choosing which recipes to try. 