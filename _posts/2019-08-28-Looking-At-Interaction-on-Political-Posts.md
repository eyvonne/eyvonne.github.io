---
layout: post
title: Looking at Interactions on Political Posts
subtitle: A look at Comment count versus Shares on Facebook
tags: [Facebook, Politics, Social Media]
comments: true
---

### Abstract

Mainstream pages get much higher comment counts per the number of shares they got. The Data was collected from a Buzzfeed article on fact-checking Facebook political posts posted October 20,2016 ([original article](https://www.buzzfeednews.com/article/craigsilverman/partisan-fb-pages-analysis#.ia1QB2KJl), data on [github](https://github.com/BuzzFeedNews/2016-10-facebook-fact-check/blob/master/data/facebook-fact-check.csv)). It was then analyzed for the number of interactions (measured in reactions, shares, and comments) compared to the pages that had posted articles. All other categorical data was removed. Missing values for all interactions were considered to be 0. The graph was constructed using a 95% confidence interval. After visual inspection T-Testing was done with 99% confidence to confirm graphical findings. 

### Introduction

This data was pulled from a Buzzfeed article on fact checking Facebook pages. The original article was posted on October 20th, 2016. The data was originally collected to look at the truthfulness. In this article the overall engagement versus page type will be considered most. 

### Cleaning

First the data is loaded into a dataframe and a few general features are examined.




```
import pandas as pd
import matplotlib.pyplot as plt
from scipy import stats
```


```
url='https://raw.githubusercontent.com/BuzzFeedNews/2016-10-facebook-fact-check/master/data/facebook-fact-check.csv'
df=pd.read_csv(url)
print(df.columns.tolist())
```

    ['account_id', 'post_id', 'Category', 'Page', 'Post URL', 'Date Published', 'Post Type', 'Rating', 'Debate', 'share_count', 'reaction_count', 'comment_count']



```
df.shape
```




    (2282, 12)



The Data is composed of just under 2300 posts, with 12 features measured. These articles are rated for their factual content, or lack thereof. This analysis is only concerned with factual content so all those items without factual content were removed. Of these, the only that are needed are the Category, Page, share_count, reaction_count, and comment_count. 


```
df=df[df['Rating']!='no factual content']
cols=['Category', 'Page', 'share_count', 'reaction_count', 'comment_count']
df=df[cols]
df.shape
```




    (2018, 5)



After removing posts with no factual content we are left with just over 2000 posts.



There are very few NaN values left to handle. There is no original documentation about what these missing values represent, however only comment count goes all the way to zero, implying that the null values represent a lack of interaction. For the purposes of analysis therefor all of the NaN values will 
filled with 0's. 


```
df.isna().sum()
```




    Category           0
    Page               0
    share_count       41
    reaction_count     2
    comment_count      2
    dtype: int64




```
df=df.fillna(0)
```

For analysis two additional features were created for the difference between the reactions and comments and the shares and comments. 


```
df['reactionCommentDiff']=df['reaction_count']-df['comment_count']
df['shareCommentDiff']=df['share_count']-df['comment_count']
```


```
df.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>share_count</th>
      <th>reaction_count</th>
      <th>comment_count</th>
      <th>reactionCommentDiff</th>
      <th>shareCommentDiff</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>2018.000000</td>
      <td>2018.000000</td>
      <td>2018.000000</td>
      <td>2018.000000</td>
      <td>2018.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>2145.583746</td>
      <td>3507.310208</td>
      <td>390.895441</td>
      <td>3116.414767</td>
      <td>1754.688305</td>
    </tr>
    <tr>
      <th>std</th>
      <td>13220.686496</td>
      <td>12087.182397</td>
      <td>1143.419917</td>
      <td>11252.480803</td>
      <td>12339.315813</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>-1261.000000</td>
      <td>-13510.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>21.000000</td>
      <td>144.000000</td>
      <td>39.000000</td>
      <td>68.250000</td>
      <td>-73.750000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>80.000000</td>
      <td>509.000000</td>
      <td>130.000000</td>
      <td>305.000000</td>
      <td>-5.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>517.500000</td>
      <td>1990.750000</td>
      <td>365.750000</td>
      <td>1573.000000</td>
      <td>235.750000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>347294.000000</td>
      <td>293333.000000</td>
      <td>32419.000000</td>
      <td>260914.000000</td>
      <td>314875.000000</td>
    </tr>
  </tbody>
</table>
</div>



The data will be examined in terms of the each page and their mean interaction counts. To simplify the graphing the math was done ahead of time and placed into a new data table, and the confidence intervals were placed into lists.  



```
pages=['Occupy Democrats', 'Addicting Info', 'The Other 98%','Politico', 'CNN Politics', 'ABC News Politics', 'Right Wing News', 'Eagle Rising', 
       'Freedom Daily']
pagesDF=[df[df['Page']==page].reset_index(drop=True) for page in pages]

meansDF=pd.DataFrame(columns=['Page','AvgReaction', 'AvgComment','AvgShare', 'AvgReactComment', 'AvgShareComment'])
for i in range(len(pages)):
  pageDF=df[df['Page']==pages[i]]
  meansDF=meansDF.append({'Page':pages[i],'AvgReaction':pageDF['reaction_count'].mean(),'AvgComment':pageDF['comment_count'].mean(),'AvgShare':pageDF['share_count'].mean(),'AvgReactComment':pageDF['reactionCommentDiff'].mean(),'AvgShareComment':pageDF['shareCommentDiff'].mean()}, ignore_index=True)
meansDF=meansDF.set_index('Page')
```


```
def CI(data):
  mean, _, _=stats.bayes_mvs(data, alpha=.95)
  return (mean[1][1]-mean[0])
commentMargin=[CI(pagesDF[i]['comment_count']) for i in range(9)]
reactionMargin=[CI(pagesDF[i]['reaction_count']) for i in range(9)]
shareMargin=[CI(pagesDF[i]['share_count']) for i in range(9)]
```

## Graphing the Data

The data is placed into a graph with the y-axis scaled logarithmically. 


```
fig=plt.figure(figsize=(12,7))
ax=fig.add_subplot(111)
react=meansDF.AvgReaction.plot(kind='bar', color='blue', ax=ax, width=.3, position=-.5,yerr=reactionMargin, label='Average Reaction Count')
share=meansDF.AvgShare.plot(kind='bar', color='purple', ax=ax, width=.3, position=1.5, yerr=shareMargin, label='Average Share Count')
comment=meansDF.AvgComment.plot(kind='bar', color='green', ax=ax, width=.3, position=0.5, yerr=commentMargin, label='Average Comment Count')
plt.xlim(left=-.5, right=8.5)
ax.set_ylabel('Average Interaction Count')
fig.suptitle('Looking at Page versus Post Interactions', fontweight='bold', fontsize=25)
fig.set_facecolor('grey')
fig.legend(loc=(.74,.805))
plt.yscale('log')
plt.xlabel("")
ax.text(-.1,1.5, '|----------Left-----------|', fontsize=15)
ax.text(2.9,1.5, '|-----"Mainstream"-----|', fontsize=15)
ax.text(6,1.5, '|----------Right----------|', fontsize=15)
plt.show()
```


![png](https://raw.githubusercontent.com/eyvonne/eyvonne.github.io/master/img/2019_08_28_Looking_At_Interaction_on_Political_Posts_files/2019_08_28_Looking_At_Interaction_on_Political_Posts_18_0.png)


The graph shows that the left-leaning pages get far more interaction than the others, with the right leaning pages coming in second. This disparity came from those pages having substantially more subscribers than the others. Interesting to note is that the comments on the mainstream pages and right leaning pages while all other interactions are higher on the right leaning pages. This suggests that people interacting with these pages are more willing to put the effort into stating their opinions. The error bars reveal the spread of the data. Occupy Democrats had far more extreme articles that were able to reach a very large audience and get tons of interactions, which introduced extra error for that page. 

To confirm the visual assumption from the graph a series of two sample t-tests was run with a 99% confidence ranking. For any two pages the null hypothesis is that there is no significant difference between them. For the vast majority of page pairs the null hypothesis is rejected so for sake of clarity only those which are not significantly different from one another are listed. 


```
for i in range(9):
  for j in range(i+1,9):
    t, p=stats.ttest_ind(pagesDF[i]['shareCommentDiff'],pagesDF[j]['shareCommentDiff'])
    if p>.01:
      print('The Null Hypothesis is accepted for', pages[i],'and',pages[j])
```

    The Null Hypothesis is accepted for Occupy Democrats and The Other 98%
    The Null Hypothesis is accepted for Addicting Info and Right Wing News
    The Null Hypothesis is accepted for Addicting Info and Eagle Rising
    The Null Hypothesis is accepted for Politico and CNN Politics
    The Null Hypothesis is accepted for Politico and ABC News Politics
    The Null Hypothesis is accepted for CNN Politics and ABC News Politics
    The Null Hypothesis is accepted for Right Wing News and Eagle Rising



```
for i in range(9):
  for j in range(i+1, 9):
    t, p=stats.ttest_ind(pagesDF[i]['reactionCommentDiff'],pagesDF[j]['reactionCommentDiff'])
    if p>.01:
      print('The Null Hypothesis is accepted for', pages[i],'and',pages[j])
```

    The Null Hypothesis is accepted for Addicting Info and Right Wing News
    The Null Hypothesis is accepted for Addicting Info and Freedom Daily
    The Null Hypothesis is accepted for Politico and Eagle Rising
    The Null Hypothesis is accepted for CNN Politics and Eagle Rising


### Conclusion
The most interesting thing about this graph is the relationship between comment count and the reaction/share count for the mainstream pages. The ratio has to be considered because of the disparity in the number of subscribers each page has. For mainstream pages the ratio of shares to other forms of interaction is much higher.

The T-Testing revealed that while the difference between reactions and comments between the right and mainstream wasn't significant, the difference in shares and comments was. This is fascinating because it implies that viewers of mainstream pages are more likely to have enough of an opinion to put the effort into writing a comment but does not make people care enough to want their friends to see it. 
