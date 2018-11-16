# PHBS_MLF_2018

## Project
Predict the outcome of elections in the Swiss Canton of Zurich 

## Members
Joel Gysel, 221802010272

## 1 Introduction 
Switzerland is divided into 26 states called "Cantons". Zurich is the most populous canton of Switzerland and it consists of 166 municipalities. Every four years, elections take place in Zurich. In Switzerland there are typically two or three left-wing parties and three centrist or right-wing parties. 

The aim of this project is first to predict the share of votes that the three left-wing parties combined in Zurich will reach in the next election and second to define a threshold and assess the effectiveness of machine learning classifiers in predicting wheather the left-wing parties will reach a voteshare above this threshold or not. 

<img src="zurich_sp_1995.png" width="650" height="700">

## 2 Data description
All relevant data can be retrieved from https://opendata.swiss/de/. The election data starts in 1995 and covers six elections. However, most explanatory variables are only available for the elections 2003, 2007, 2011 and 2015, therefore we focuse our analysis on those four elections.

The explained variable is voteshare, which consists of the combined party strength (share of votes) of the three parties "Sozialdemokratische Partei (SP)", "Grüne Partei (GP)" and "Alternative Liste (AL)". Those three parties are forming the left wing in the Kanton of Zurich. 

<img src="images/position parteien.png" width="500" height="400">

The explanatory variables are:

Feature |Description
-------|---------
children_ratio	| Population between 0 and 18 years divided by population between 18 and 64
participation	| Participation in election in %
foreigners	| Share of foreign people compared to whole population 
old_ratio	| Population above 65 divided by population between 18 and 64
social_aid	| Number of unemployed people who receive social aid
income	| Median income 
wealth	| Median wealth (net assets) 
population_density	| Population density within a district
voteshare_past | Share of votes of left-wing parties in last election


Not included in the data frame but still a very important variable is the variable lucerne. As you can see in the graph below, the outcome of the Zurich election follows usually the trend of elections prior to the Zurich election. The canton of Lucerne has its election just a few month before the Zurich election takes place and if the left-wing parties are losing votes in Lucerne, it is very likely that the same will happen in the Zurich elections. By including the percentage changes in voteshare from Lucerne, we aim to further improve our model. 


<img src="images/kantonale wahlen.png" width="550" height="300">



## 3 Predict continuous output variable 
### Regression
#### Regression with absolute values
Before we start running regressions with multiple explanatory, we want to run a very sparse model with just one explanatory variable: The outcome of the last election. This sparse model will further be called "univariate". This simple model helps to demonstrate the problem with absolute values. 

<img src="images/r2 absolute values.png" width="450" height="300">

The graph above is used to evaluate the performance of this simple regression and of all future models. The y-axis shows the R^2 in case of a regression or the score in case of a classification. The x-axis shows, what we evaluate. The 2003 data point will always be the performance of the model within the train-dataset (2003). The years 2007 to 2015 represent a cross-validation with either fitting on just the precedent election or fitting based on all precedent elections. 

Let us first have a look at the blue line: Here we fit the outcome of the 2003 election to the outcome of the 1999 election and use the coefficients to make an out-of-sample prediction for the 2007 election (same mechanism for the following years). This model performs very poorly with even a negative R^2 in 2007. Due to the very poor performance, I was tempted to test a second model: I fitted the 2003 outcome on the 2003 outcome such that I had a model with an intercept of zero and and a beta coefficient of 1 (this model will be called zero/one). Testing this model in a forecast basically means to use the outcome of the last elections as the prediction for the next election. This model, even though performing a bit better, still performs very bad, so past elections do not seem to contain a lot of information about future elections (similar as financial data). 

#### Regression with relative values
To improve my model, I now start working with relative values. The idea: if I transform my data, models will likely perform much better. I can apply regressions and classifiers to relative data and then transform the data back to absolute values such that I get a better forecast of the outcome of elections. 

If I would transform the data s.t. it follows standard normal distribution, I would have problems transforming the data back since I have no information about the variance. However, if I only deduct the mean from "voteshare", I can apply regressions and classifiers to relative data and then add the mean and the percentage change from the election of Lucerne to the predicted values in order to get back to absolute values (and hopefully better absolute values). 

However, if I only deduct the mean from the data, I have to make sure that the variance is comparable in each election year, otherwise my models will be missleading. To test the null-hypothesis of equal variance, I apply the fligner-killeen test to the voteshare of my four election years: 

fligner-killeen test|-
--------------------|---------
test statistic      | 2.067
P-value	            | 0.5585

Luckily, we get a p-value of over 0.55 which means that we cannot reject the null-hypothesis of equal variance in the election years. Given this result, I am now deducting the mean within each election year. The two histogramms below show the data before and after the transformation: 

<img src="images/voteshares.png" width="300" height="250">
<img src="images/voteshares demeaned.png" width="300" height="250">

Now I run the same regressions again as before, but this time I use relative values: 

<img src="images/r2 relative values.png" width="450" height="300">

The accuracy of my model is much better now. This means: If I can predict the change in mean by making use of election data from the canton of Lucerne, it should be possible to improve my model. A second observation of the above graph is that the zero/one model is still performing better than a fitted model. Next, I will try to improve the fitted model by making the fit not just based on the last election but on all last elections, which means that my dataset for fitting is growing over time. 

<img src="images/r2 relative values sigma.png" width="450" height="300">

As we can see in the graph, in 2003 (train accuracy) and 2007 (first out-of-sample-prediction), this approach will lead to the same results as the fitted model before since the dataset only starts to be bigger from 2011 onward. For 2011 and 2015, this fitted model performs better than before but still worse than the zero/one model. 


#### Regression with multiple explanatory variables

Next, I will try to run a multivariate regression that includes all explanatory variables. I will provide a short example to provide the idea behind the multiple regression: If during the fit stage our model learns, that a higher share of foreign people leadst to a significantly higher voteshare for the left-wing parties, then it will predict a higher voteshare if the percentage of foreigners increased in a community from one election to the other, making the model more accurate. Let us see, how well a multiple regression performs: 

<img src="images/r2 selected vs multi.png" width="450" height="300">

In case of our zero/one model, there is almost no difference in the R^2 of a multivariate model compared to our univariate model before with voteshare_past as the only explanatory variable. In case of our model "last election" that uses coefficients from a fit of secondlast to last election, the multivariate model performs even worse than the univariate model. Maybe this result is due to overfit. Let us asses the heatmap of all our coefficients: 

<img src="images/heatmap.png" width="500" height="500">

The last row is the output variable (voteshare_demeaned). As we would expect, voteshare_past_demeaned has by far the highest correlation with voteshare_demeaned. Other factors such as median wealth, percentage of foreigners and population density seem to have some explanatory power as well. We use those four factors and remove all other factors for a more sparse model. This new model with only four factors shows the following performance: 

<img src="images/r2 selected vs multi.png" width="450" height="300">

As we can see, our results have not really improved compared to the full multivariate model. 

#### Regression: conclusion
Even though we have tested a lot of different models so far, it seems not to be possible to create a model that performs better than our simple zero/one model where I just use the outcome of the last election as the prediction for the next election. Given this result, I will now use the predicted values of the univariate zero/one model and add the mean back again plus a correction factor from the outcome of the election in Lucerne and test the performance of this model. The results can be found in the table below: 

 -- | R^2 before |R^2 after
----|---------|---------
2007|	-1.237 |	0.544
2011|	0.665 |	0.468
2015|	0.382 |	0.855

Our new R^2 of the predicted absolute values is still well below our R^2 with relative data. This is due to the fact that the outcome of the preceeding Lucerne election is not a precise forecast for the outcome of the Zurich election but rather an approximation. Our R^2 is lower because it is impossible to predict the change in mean exactly. Nevertheless, with our transformation to relative data and our backtransformation plus correction, we can overall reach a higher R^2 than before.


### Decision tree Regression 
A different method to predict a continuous output variable is the decision tree regression. I will shortly test this model in order to assess if it has a better performance than our best performing regression model (zero/one). 

<img src="images/r2 decision tree.png" width="450" height="300">

As we can see, our decision tree regression performs worse than the zero/one model. The answer to this worse performance is found in the next graph: 

<img src="images/decision tree regression.png" width="400" height="300">

In our case, we have an almost linear relationship between the outcome of the last vote and the outcome of the current vote. As the plot shows, it is not suitable to use a decision tree regression in case of an almost perfect linear relationship. 


## 5 Apply ML method to discrete output variable 
As stated in the introduction, we will now assess the effectiveness of classifiers for a voteshare threshold. Threshold classifications for predicting the outcome of elections can be important if for example we want to predict if a party will reach a majority in a ceratin area (e.g. president elections in the U.S.) or if there is a minimum voteshare that a party must reach in order to receive seats in the parlament. In Zurich none of those two examples is especially relevant, therefore we define the threshold just as the median party strength of the left-wing parties. By defining the threshold this way, we avoid problems with an imbalanced dataset.  

### Logistic regression 
<img src="decision boundaries log reg.png" width="400" height="300">

Illustration of the problem of weak explanatory variables: Feature "foreigners" has almost no impact on decision boundary. Furthermore, the error of missclassified samples seems to nonsystematic (= random). 

<img src="accuracy logistic regression.png" width="400" height="300">

Multivariate regression performs slightly better than univariate regression. Positive trend is probably random. 

### Decision Tree
Optimal depth: 

<img src="depth decision tree.png" width="400" height="300">

A depth of 5 leaves seems to be optimal for the decision tree

Accuracy: 

<img src="accuracy decision tree.png" width="400" height="300">

Multivariate regression is slightly better than univariate regression. However, performance over time is decreasing and overall performance is bad. 


### KNN
Optimal depth: 
<img src="depth KNN.png" width="400" height="300">

Difficult to determine optimal depth, needs some further testing through cross-validation. Our choice for now: 5 neighbours. 

Accuracy: 
<img src="accuracy KNN.png" width="400" height="300">

As expected is the univariate model in this case better than the multivariate one. 

## 6 Conclusion 
* Predicting elections is similar as predicting stock prices: Today's outcome of elections is the best predictor for future outcomes and once we account for the change in the mean, differences in voteshares are almost random. 
  * Possible explanations: 
      * Humains are involved and humains behave irrational and unpredictable 
      * Shares of votes is largely dependent on candidates. A voter's attitude towards a candidate is very difficult to measure 
* A multivariate model can still improve the model, even if only by a small percentage 
  * In case of a continuous output variable, we should prefer the linear regression model over the decision tree regression
  * In case of a discrete output variable, we should chose the logistic regression: Best accuracy and best robustness
* Further explanatory should be tested for their predictive power
  * Possible variables: 
    * Sentiment analysis in social media 
    * Some explanatory variables on candidates


