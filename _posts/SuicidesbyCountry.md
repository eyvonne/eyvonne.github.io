<table>

<tbody>

<tr>

<td>

#### Introduction:

Worldwide in 2016 suicide claimed 16 men per 100,000 people, and 7 women per 100k. This model uses data from 1985 through 2015 found on [Kaggle](https://www.kaggle.com/russellyates88/suicide-rates-overview-1985-to-2016). The model uses country demographic information to attempt to predict the number of suicides for men and women in given age brackets. This data does not take the modern queering of gender into account, which has a high impact on suicide rates worldwide. Global trends are difficult to track, some countries have a distinctly positive trend, others have a negative trend, and still others have no trends visible at all. It was originally assumed that the Human Development Index would be a good metric to predict suicide rates. As the HDI goes up it would make sense for the suicide rate to drop.

</td>

<td>

![](https://raw.githubusercontent.com/eyvonne/eyvonne.github.io/master/img/SuicidesByCountry/ChileSuicideRates.png) 

![](https://raw.githubusercontent.com/eyvonne/eyvonne.github.io/master/img/SuicidesByCountry/BeliezeSuicideRates.png)

![](https://raw.githubusercontent.com/eyvonne/eyvonne.github.io/master/img/SuicidesByCountry/IcelandSuicideRates.png)

</td>

</tr>

<tr>

<td>

#### Data Cleaning:

 The annual GDP was listed originally as a string with commas that needed to be removed and formatted as ints. The original data also included a column that had the year and country combined, which wasn’t useful for this model so it was dropped. The HDI was the only feature with missing values. To impute these values a custom function was written that used the average HDI for the country. There were some countries which had no HDI listed, rather than guess these countries were dropped. It was originally planned to use the total number of suicides as the target vector but after building an initial model it was decided it would make more sense to predict the suicides/100k population. This normalization took the population from being the largest impact on the final prediction to being one of the smallest, focusing instead on the sex and HDI. 

</td>

</tr>

<tr>

<td>

#### Modeling:

Originally an XGBoost regressor was fit to the data, but it was found that the model would predict values of less than zero, which didn’t make sense for this dataset. By using a random forest the range on the output was limited to being similar to that of the input, meaning that the predictions never said that a country would experience a negative number of suicides. These two shap plots are made using the same data, just using the different models. The actual suicides per 100k for this data was 3.77.

</td>

</tr>

<tr>

<td colspan="2">

XGB:

![](https://raw.githubusercontent.com/eyvonne/eyvonne.github.io/master/img/SuicidesByCountry/XGBShapPlot.png)

</td>

</tr>

<tr>

<td colspan="2">

Random Forest: 

 ![](https://raw.githubusercontent.com/eyvonne/eyvonne.github.io/master/img/SuicidesByCountry/RandomForestRegressor.png)

</td>

</tr>

</tbody>

</table>

<table>

<tbody>

<tr>

<td>

Results:

 The Random Forest model was able to achieve a mean absolute error of 8.13 suicides per 100k people. This beats the mean baseline score of 13.09\. Feature permutation importance of this shows that the sex is the most important feature as shown in the ELI5 chart at right. The partial dependence for sex is relatively boring because it only has two categories, men on average have a higher rate, while women a lower rate. The HDI for the year produces a much more interesting partial dependence plot, shown below (left).

</td>

<td>![](https://raw.githubusercontent.com/eyvonne/eyvonne.github.io/master/img/SuicidesByCountry/Eli5.pnghttps://raw.githubusercontent.com/eyvonne/eyvonne.github.io/master/img/SuicidesByCountry/Eli5.png)</td>

</tr>

<tr>

<td>![](https://raw.githubusercontent.com/eyvonne/eyvonne.github.io/master/img/SuicidesByCountry/HDIpdp.png)</td>

<td>

This plot shows that as the Human development index goes up, on average so does the rate of suicide. The Human Development Index is a score assigned based on the life expectancy, knowledge, and the standard of living. It would make sense for this to therefore indicate a lower rate of suicide.The around of feature engineering that has to be done therefore is concerning. Wikipedia mentions that the 2010 Human Development Report introduced a new metric, the income adjusted HDI, which takes inequality into account and is now considered the more accurate measure. While the new metric is not based on the suicide the inequality not measured in the HDI could inform the increased suicide rate. The spike around .8 is also interesting because the mean and median of the data is at about .77, I’m curious if that spike had to happen from the feature engineering that I did. 

</td>

</tr>

</tbody>

</table>
