
### Abstract

The Data was collected from a Buzzefeed article on fact-checking Facebook political posts. It was then analyzed for the number of interactions (measured in reactions, shares, and comments) compared to the pages that had posted articles. All other categorical data was removed. Missing values for all interactions were considered to be 0. The graph was constructed using a 95% confidence interval. It was found that mainstream pages get much higher comment counts per the number of reactions and shares they got. 

### Introduction

This data was pulled from a Buzzfeed article on fact checking Facebook pages. The original article can be found [here](https://www.buzzfeednews.com/article/craigsilverman/partisan-fb-pages-analysis#.ia1QB2KJl). The data has been published and is available on [github](https://github.com/BuzzFeedNews/2016-10-facebook-fact-check/blob/master/data/facebook-fact-check.csv). The original article was posted on October 20th, 2016. The data was originally collected to look at the truthfulness, in this article the overall engagement versus page type will be considered most.

### Cleaning

First the data is loaded into a dataframe and a few general features are examined




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

There are very few Na values left to handle. There is no original documentation about what these missing values represent, however only comment count goes all the way to zero, implying that the null values represent a lack of interaction. For the purposes of analysis therefor all of the NaN values will be filled with 0's. 


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
      <td>1977.000000</td>
      <td>2016.000000</td>
      <td>2016.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>2190.079919</td>
      <td>3510.789683</td>
      <td>391.283234</td>
    </tr>
    <tr>
      <th>std</th>
      <td>13353.490029</td>
      <td>12092.674184</td>
      <td>1143.920876</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.000000</td>
      <td>2.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>22.000000</td>
      <td>144.000000</td>
      <td>39.750000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>84.000000</td>
      <td>509.500000</td>
      <td>130.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>557.000000</td>
      <td>1992.250000</td>
      <td>366.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>347294.000000</td>
      <td>293333.000000</td>
      <td>32419.000000</td>
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
ax.text(-.1,1.5, '-----------Left-----------', fontsize=15)
ax.text(2.9,1.5, '-----"Mainstream"-----', fontsize=15)
ax.text(6,1.5, '----------Right----------', fontsize=15)
plt.show()
```


![png](2019_08_23_Looking_At_Interaction_on_Political_Posts-2_files/2019_08_23_Looking_At_Interaction_on_Political_Posts-2_16_0.png)


The graph shows that the left leaning pages get far more interaction than the others, with the right leaning pages coming in second. This disparity came from those pages have dramatically more subscribers than the others. Interesting to note is that the comments on the mainstream pages and right leaning pages while all other interactions are higher on the right leaning pages. This suggests that people interacting with these pages are more willing to put the effort into stating their opinions. The error bars primarily reveal the spread of the data more than anything, Occupy Democrats had far more extreme articles that were able to reach a very large audience and get tons of interactions, which introduced extra error for that page. 


### Takeaway
The most interesting thing about this graph is the relationship between comment count and the reaction/share count for the mainstream pages. The ratio has to be considered because of the disparity in the number of subscribers each page has. For mainstream pages the ratio of shares to other forms of interaction is much higher. 
