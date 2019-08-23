---
layout: post
title: Looking At Interaction on Political Posts
subtitle: Data storytelling project by Eyve Geordan
tags: [Social Media, Politics]
comments: true
---

### Introduction

This data was pulled from a Buzzfeed article on fact checking Facebook pages. The original article can be found [here](https://www.buzzfeednews.com/article/craigsilverman/partisan-fb-pages-analysis#.ia1QB2KJl). The data has been published and is available on [github](https://github.com/BuzzFeedNews/2016-10-facebook-fact-check/blob/master/data/facebook-fact-check.csv). The original article was posted on October 20th, 2016. The data was orriginally collected to look at the truthfulness, in this article the overall engagement versus page type will be considered most.

### Cleaning

First the data is loaded into a dataframe and the head is examined.




```
import pandas as pd
import matplotlib.pyplot as plt
from scipy import stats
```


```
url='https://raw.githubusercontent.com/BuzzFeedNews/2016-10-facebook-fact-check/master/data/facebook-fact-check.csv'
df=pd.read_csv(url)
df.head()
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
      <th>account_id</th>
      <th>post_id</th>
      <th>Category</th>
      <th>Page</th>
      <th>Post URL</th>
      <th>Date Published</th>
      <th>Post Type</th>
      <th>Rating</th>
      <th>Debate</th>
      <th>share_count</th>
      <th>reaction_count</th>
      <th>comment_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>184096565021911</td>
      <td>1035057923259100</td>
      <td>mainstream</td>
      <td>ABC News Politics</td>
      <td>https://www.facebook.com/ABCNewsPolitics/posts...</td>
      <td>2016-09-19</td>
      <td>video</td>
      <td>no factual content</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>146.0</td>
      <td>15.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>184096565021911</td>
      <td>1035269309904628</td>
      <td>mainstream</td>
      <td>ABC News Politics</td>
      <td>https://www.facebook.com/ABCNewsPolitics/posts...</td>
      <td>2016-09-19</td>
      <td>link</td>
      <td>mostly true</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>33.0</td>
      <td>34.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>184096565021911</td>
      <td>1035305953234297</td>
      <td>mainstream</td>
      <td>ABC News Politics</td>
      <td>https://www.facebook.com/ABCNewsPolitics/posts...</td>
      <td>2016-09-19</td>
      <td>link</td>
      <td>mostly true</td>
      <td>NaN</td>
      <td>34.0</td>
      <td>63.0</td>
      <td>27.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>184096565021911</td>
      <td>1035322636565962</td>
      <td>mainstream</td>
      <td>ABC News Politics</td>
      <td>https://www.facebook.com/ABCNewsPolitics/posts...</td>
      <td>2016-09-19</td>
      <td>link</td>
      <td>mostly true</td>
      <td>NaN</td>
      <td>35.0</td>
      <td>170.0</td>
      <td>86.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>184096565021911</td>
      <td>1035352946562931</td>
      <td>mainstream</td>
      <td>ABC News Politics</td>
      <td>https://www.facebook.com/ABCNewsPolitics/posts...</td>
      <td>2016-09-19</td>
      <td>video</td>
      <td>mostly true</td>
      <td>NaN</td>
      <td>568.0</td>
      <td>3188.0</td>
      <td>2815.0</td>
    </tr>
  </tbody>
</table>
</div>




```
df.shape
```




    (2282, 12)



The Data is composed of just under 2300 posts, with 12 features measured. Of these, the only that are needed are the Category, Page, share_count, reaction_count, and comment_count. 


```
cols=['Category', 'Page', 'share_count', 'reaction_count', 'comment_count']
df=df[cols]
```

Taking a look at the data that we have left there are very few Na values to handle. There is no original documentation about what these missing values represent, however only comment count goes all the way to zero, implying that the null values represent a lack of interaction. For the purposes of analysis therefor all of the NaN values will be filled with 0's. 


```
df.isna().sum()
```




    Category           0
    Page               0
    share_count       70
    reaction_count     2
    comment_count      2
    dtype: int64




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
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>2.212000e+03</td>
      <td>2280.000000</td>
      <td>2280.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>4.044816e+03</td>
      <td>5364.284649</td>
      <td>516.102193</td>
    </tr>
    <tr>
      <th>std</th>
      <td>2.983192e+04</td>
      <td>19126.544561</td>
      <td>3569.355445</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.000000e+00</td>
      <td>2.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>2.400000e+01</td>
      <td>149.000000</td>
      <td>37.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>9.600000e+01</td>
      <td>545.500000</td>
      <td>131.500000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>7.390000e+02</td>
      <td>2416.750000</td>
      <td>390.250000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>1.088995e+06</td>
      <td>456458.000000</td>
      <td>159047.000000</td>
    </tr>
  </tbody>
</table>
</div>




```
df=df.fillna(0)
```

The data will be examined in terms of the each page and their mean interaction counts. To simplify the graphing all the math was done ahead of time and placed into a new data table, and all the confidence intervals were placed into lists.  



```
pages=['Occupy Democrats', 'Addicting Info', 'The Other 98%','Politico', 'CNN Politics', 'ABC News Politics', 'Right Wing News', 'Eagle Rising', 
       'Freedom Daily']
pagesDF=[df[df['Page']==page].reset_index(drop=True) for page in pages]

meansDF=pd.DataFrame(columns=['Page','AvgReaction', 'AvgComment','AvgShare'])
for i in range(len(pages)):
  pageDF=df[df['Page']==pages[i]]
  meansDF=meansDF.append({'Page':pages[i],'AvgReaction':pageDF['reaction_count'].mean(),'AvgComment':pageDF['comment_count'].mean(),'AvgShare':pageDF['share_count'].mean()}, ignore_index=True)
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

The data is placed into a graph with two y-axes. On the left are the reaction and share counts while on the right is the comment count. This is because the comment count is dramatically lower than the other two and needs a different scale to accurately show. 


```
fig=plt.figure()
ax=fig.add_subplot(111)
#ax2=ax.twinx()
react=meansDF.AvgReaction.plot(kind='bar', color='blue', ax=ax, width=.3, position=0,yerr=reactionMargin, label='Average Reaction Count')
share=meansDF.AvgShare.plot(kind='bar', color='purple', ax=ax, width=.3, position=2, yerr=shareMargin, label='Average Share Count')
comment=meansDF.AvgComment.plot(kind='bar', color='green', ax=ax, width=.3, position=1, yerr=commentMargin, label='Average Comment Count')
plt.xlim(left=-.7, right=8.7)
ax.set_ylabel('Average Interaction Count')
#ax2.set_ylabel('Comment Count')
fig.suptitle('Looking at Page versus Post Interactions')
fig.set_facecolor('grey')
fig.legend(loc=(.5,.75))
plt.yscale('log')
plt.show()
```


![jpeg](hello_world.jpeg)


The graph shows that the left leaning pages get far more interaction than the others, with the right leaning pages coming in second. Interesting to note is that the comments on the mainstream pages and right leaning pages while all other interactions are higher on the right leaning pages. This sugests that people interacting with these pages are more willing to put the effort into stating ther opinions. The error bars primarily reveal the spread of the data more than anything, Occupy Democrats had far more extreme articles that were able to reach a very large audience and get tons of interactions, which introduced extra error for that page. 
