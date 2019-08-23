
Introduction

This data was pulled from a Buzzfeed article on fact checking Facebook pages. The original article can be found at: [link]. The data has been published and is available on github at: [link]. The original article was posted on October 20th, 2016. The data was orriginally collected to look at the truthfulness, in this article the overall engagement versus page type will be considered most.

Cleaning

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

## graphing the data

The data is placed into a graph with two y-axes. On the left are the reaction and share counts while on the right is the comment count. This is because the comment count is dramatically lower than the other two and needs a different scale to accurately show. 


```
fig=plt.figure()
ax=fig.add_subplot(111)
#ax2=ax.twinx()
react=meansDF.AvgReaction.plot(kind='bar', color='blue', ax=ax, width=.3, position=0,yerr=reactionMargin, label='Average Reaction Count')
share=meansDF.AvgShare.plot(kind='bar', color='purple', ax=ax, width=.3, position=2, yerr=shareMargin, label='Average Share Count')
comment=meansDF.AvgComment.plot(kind='bar', color='green', ax=ax, width=.3, position=1, yerr=commentMargin, label='Average Comment Count')
plt.xlim(left=-.7, right=8.7)
ax.set_ylabel('Reaction or Share Count')
ax2.set_ylabel('Comment Count')
fig.suptitle('Looking at Page versus Post Interactions')
fig.set_facecolor('grey')
fig.legend(loc=(.5,.75))
plt.yscale('log')
plt.show()
```


![png](DataStorytellingWriteUp_files/DataStorytellingWriteUp_14_0.png)


The graph clearly shows that the left leaning pages get far more interaction than the others, with the right leaning pages coming in second. Interesting to note is that the comments on the mainstream pages and right leaning pages while all other interactions are higher on the right leaning pages. This sugests that people interacting with these pages are more willing to put the effort into stating ther opinions. The error bars primarily reveal the spread of the data more than anything, Occupy Democrats had far more extreme articles that were able to reach a very large audience and get tons of interactions, which introduced extra error for that page. 

### playground. There is a minimum viable product above this line



```

```


```
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
      <th>Category</th>
      <th>Page</th>
      <th>share_count</th>
      <th>reaction_count</th>
      <th>comment_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>mainstream</td>
      <td>ABC News Politics</td>
      <td>0.0</td>
      <td>146.0</td>
      <td>15.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>mainstream</td>
      <td>ABC News Politics</td>
      <td>1.0</td>
      <td>33.0</td>
      <td>34.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>mainstream</td>
      <td>ABC News Politics</td>
      <td>34.0</td>
      <td>63.0</td>
      <td>27.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>mainstream</td>
      <td>ABC News Politics</td>
      <td>35.0</td>
      <td>170.0</td>
      <td>86.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>mainstream</td>
      <td>ABC News Politics</td>
      <td>568.0</td>
      <td>3188.0</td>
      <td>2815.0</td>
    </tr>
  </tbody>
</table>
</div>




```
pagesDF[0][['Page','share_count','reaction_count', 'comment_count']].T.head()
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
      <th>1147</th>
      <th>1148</th>
      <th>1149</th>
      <th>1150</th>
      <th>1151</th>
      <th>1152</th>
      <th>1153</th>
      <th>1154</th>
      <th>1155</th>
      <th>1156</th>
      <th>1157</th>
      <th>1158</th>
      <th>1159</th>
      <th>1160</th>
      <th>1161</th>
      <th>1162</th>
      <th>1163</th>
      <th>1164</th>
      <th>1165</th>
      <th>1166</th>
      <th>1167</th>
      <th>1168</th>
      <th>1169</th>
      <th>1170</th>
      <th>1171</th>
      <th>1172</th>
      <th>1173</th>
      <th>1174</th>
      <th>1175</th>
      <th>1176</th>
      <th>1177</th>
      <th>1178</th>
      <th>1179</th>
      <th>1180</th>
      <th>1181</th>
      <th>1182</th>
      <th>1183</th>
      <th>1184</th>
      <th>1185</th>
      <th>1186</th>
      <th>...</th>
      <th>1316</th>
      <th>1317</th>
      <th>1318</th>
      <th>1319</th>
      <th>1320</th>
      <th>1321</th>
      <th>1322</th>
      <th>1323</th>
      <th>1324</th>
      <th>1325</th>
      <th>1326</th>
      <th>1327</th>
      <th>1328</th>
      <th>1329</th>
      <th>1330</th>
      <th>1331</th>
      <th>1332</th>
      <th>1333</th>
      <th>1334</th>
      <th>1335</th>
      <th>1336</th>
      <th>1337</th>
      <th>1338</th>
      <th>1339</th>
      <th>1340</th>
      <th>1341</th>
      <th>1342</th>
      <th>1343</th>
      <th>1344</th>
      <th>1345</th>
      <th>1346</th>
      <th>1347</th>
      <th>1348</th>
      <th>1349</th>
      <th>1350</th>
      <th>1351</th>
      <th>1352</th>
      <th>1353</th>
      <th>1354</th>
      <th>1355</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Page</th>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>...</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
      <td>Occupy Democrats</td>
    </tr>
    <tr>
      <th>share_count</th>
      <td>9144</td>
      <td>26696</td>
      <td>121577</td>
      <td>447</td>
      <td>17330</td>
      <td>7122</td>
      <td>14207</td>
      <td>32189</td>
      <td>12984</td>
      <td>2007</td>
      <td>5207</td>
      <td>33482</td>
      <td>3812</td>
      <td>63291</td>
      <td>0</td>
      <td>2460</td>
      <td>7338</td>
      <td>46108</td>
      <td>3910</td>
      <td>10093</td>
      <td>46081</td>
      <td>12646</td>
      <td>18396</td>
      <td>31723</td>
      <td>21884</td>
      <td>1204</td>
      <td>10654</td>
      <td>3813</td>
      <td>16922</td>
      <td>1</td>
      <td>8188</td>
      <td>3151</td>
      <td>16599</td>
      <td>10931</td>
      <td>2761</td>
      <td>31014</td>
      <td>41515</td>
      <td>27749</td>
      <td>347294</td>
      <td>8261</td>
      <td>...</td>
      <td>19481</td>
      <td>942</td>
      <td>14302</td>
      <td>261962</td>
      <td>8404</td>
      <td>6</td>
      <td>5030</td>
      <td>1166</td>
      <td>14273</td>
      <td>24356</td>
      <td>12361</td>
      <td>58940</td>
      <td>10975</td>
      <td>3483</td>
      <td>22675</td>
      <td>322630</td>
      <td>10303</td>
      <td>10269</td>
      <td>11653</td>
      <td>34023</td>
      <td>7598</td>
      <td>2678</td>
      <td>32230</td>
      <td>2</td>
      <td>12376</td>
      <td>76113</td>
      <td>34676</td>
      <td>10799</td>
      <td>8670</td>
      <td>4911</td>
      <td>5541</td>
      <td>1225</td>
      <td>7540</td>
      <td>11923</td>
      <td>0</td>
      <td>5054</td>
      <td>15508</td>
      <td>12897</td>
      <td>4891</td>
      <td>12212</td>
    </tr>
    <tr>
      <th>reaction_count</th>
      <td>13983</td>
      <td>33959</td>
      <td>54024</td>
      <td>3077</td>
      <td>18248</td>
      <td>10154</td>
      <td>33399</td>
      <td>42085</td>
      <td>19513</td>
      <td>8342</td>
      <td>10719</td>
      <td>19552</td>
      <td>11005</td>
      <td>30011</td>
      <td>30011</td>
      <td>4723</td>
      <td>13357</td>
      <td>31113</td>
      <td>6502</td>
      <td>6617</td>
      <td>55197</td>
      <td>27507</td>
      <td>95902</td>
      <td>25045</td>
      <td>17028</td>
      <td>3585</td>
      <td>13342</td>
      <td>12549</td>
      <td>47802</td>
      <td>15305</td>
      <td>14892</td>
      <td>12665</td>
      <td>22532</td>
      <td>22315</td>
      <td>8235</td>
      <td>39723</td>
      <td>67257</td>
      <td>39224</td>
      <td>293333</td>
      <td>32737</td>
      <td>...</td>
      <td>58479</td>
      <td>6556</td>
      <td>29127</td>
      <td>217848</td>
      <td>26021</td>
      <td>136057</td>
      <td>14138</td>
      <td>7585</td>
      <td>50065</td>
      <td>84835</td>
      <td>29914</td>
      <td>68577</td>
      <td>20953</td>
      <td>19265</td>
      <td>58948</td>
      <td>210016</td>
      <td>26964</td>
      <td>26345</td>
      <td>18248</td>
      <td>75077</td>
      <td>27556</td>
      <td>10312</td>
      <td>49697</td>
      <td>40931</td>
      <td>27749</td>
      <td>87538</td>
      <td>29531</td>
      <td>15652</td>
      <td>13856</td>
      <td>12859</td>
      <td>9110</td>
      <td>7503</td>
      <td>15205</td>
      <td>15228</td>
      <td>36716</td>
      <td>10271</td>
      <td>43655</td>
      <td>17085</td>
      <td>11754</td>
      <td>48240</td>
    </tr>
    <tr>
      <th>comment_count</th>
      <td>1052</td>
      <td>1762</td>
      <td>9549</td>
      <td>293</td>
      <td>2527</td>
      <td>441</td>
      <td>1192</td>
      <td>2302</td>
      <td>3204</td>
      <td>720</td>
      <td>3695</td>
      <td>1741</td>
      <td>523</td>
      <td>13510</td>
      <td>13510</td>
      <td>825</td>
      <td>561</td>
      <td>2988</td>
      <td>1339</td>
      <td>1084</td>
      <td>3296</td>
      <td>1352</td>
      <td>3754</td>
      <td>1514</td>
      <td>4888</td>
      <td>530</td>
      <td>1807</td>
      <td>1461</td>
      <td>1880</td>
      <td>322</td>
      <td>661</td>
      <td>1998</td>
      <td>2444</td>
      <td>1055</td>
      <td>401</td>
      <td>1729</td>
      <td>1264</td>
      <td>1809</td>
      <td>32419</td>
      <td>2821</td>
      <td>...</td>
      <td>1590</td>
      <td>167</td>
      <td>864</td>
      <td>11347</td>
      <td>3775</td>
      <td>1936</td>
      <td>373</td>
      <td>203</td>
      <td>1110</td>
      <td>1396</td>
      <td>1048</td>
      <td>2281</td>
      <td>525</td>
      <td>1015</td>
      <td>2222</td>
      <td>13508</td>
      <td>549</td>
      <td>714</td>
      <td>4396</td>
      <td>1598</td>
      <td>411</td>
      <td>1926</td>
      <td>2287</td>
      <td>6032</td>
      <td>1161</td>
      <td>8339</td>
      <td>4223</td>
      <td>1579</td>
      <td>3189</td>
      <td>626</td>
      <td>638</td>
      <td>172</td>
      <td>1484</td>
      <td>4217</td>
      <td>685</td>
      <td>431</td>
      <td>1669</td>
      <td>1487</td>
      <td>785</td>
      <td>1037</td>
    </tr>
  </tbody>
</table>
<p>4 rows × 209 columns</p>
</div>




```
shareDF=pd.DataFrame()
for i in range(9):
  shareDF=shareDF.append(pagesDF[i].T.loc['share_count'], ignore_index=True)
shareDF=shareDF.set_index(pd.Index(pages))
```


```
stats.ttest_ind(df['reaction_count'], df['share_count'], equal_var=False)
```




    Ttest_indResult(statistic=1.9609071201714827, pvalue=0.04996057601012408)




```
shareDF
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
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>10</th>
      <th>11</th>
      <th>12</th>
      <th>13</th>
      <th>14</th>
      <th>15</th>
      <th>16</th>
      <th>17</th>
      <th>18</th>
      <th>19</th>
      <th>20</th>
      <th>21</th>
      <th>22</th>
      <th>23</th>
      <th>24</th>
      <th>25</th>
      <th>26</th>
      <th>27</th>
      <th>28</th>
      <th>29</th>
      <th>30</th>
      <th>31</th>
      <th>32</th>
      <th>33</th>
      <th>34</th>
      <th>35</th>
      <th>36</th>
      <th>37</th>
      <th>38</th>
      <th>39</th>
      <th>...</th>
      <th>496</th>
      <th>497</th>
      <th>498</th>
      <th>499</th>
      <th>500</th>
      <th>501</th>
      <th>502</th>
      <th>503</th>
      <th>504</th>
      <th>505</th>
      <th>506</th>
      <th>507</th>
      <th>508</th>
      <th>509</th>
      <th>510</th>
      <th>511</th>
      <th>512</th>
      <th>513</th>
      <th>514</th>
      <th>515</th>
      <th>516</th>
      <th>517</th>
      <th>518</th>
      <th>519</th>
      <th>520</th>
      <th>521</th>
      <th>522</th>
      <th>523</th>
      <th>524</th>
      <th>525</th>
      <th>526</th>
      <th>527</th>
      <th>528</th>
      <th>529</th>
      <th>530</th>
      <th>531</th>
      <th>532</th>
      <th>533</th>
      <th>534</th>
      <th>535</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Occupy Democrats</th>
      <td>9144.0</td>
      <td>26696.0</td>
      <td>121577.0</td>
      <td>447.0</td>
      <td>17330.0</td>
      <td>7122.0</td>
      <td>14207.0</td>
      <td>32189.0</td>
      <td>12984.0</td>
      <td>2007.0</td>
      <td>5207.0</td>
      <td>33482.0</td>
      <td>3812.0</td>
      <td>63291.0</td>
      <td>0.0</td>
      <td>2460.0</td>
      <td>7338.0</td>
      <td>46108.0</td>
      <td>3910.0</td>
      <td>10093.0</td>
      <td>46081.0</td>
      <td>12646.0</td>
      <td>18396.0</td>
      <td>31723.0</td>
      <td>21884.0</td>
      <td>1204.0</td>
      <td>10654.0</td>
      <td>3813.0</td>
      <td>16922.0</td>
      <td>1.0</td>
      <td>8188.0</td>
      <td>3151.0</td>
      <td>16599.0</td>
      <td>10931.0</td>
      <td>2761.0</td>
      <td>31014.0</td>
      <td>41515.0</td>
      <td>27749.0</td>
      <td>347294.0</td>
      <td>8261.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>Addicting Info</th>
      <td>130.0</td>
      <td>241.0</td>
      <td>991.0</td>
      <td>1190.0</td>
      <td>1018.0</td>
      <td>375.0</td>
      <td>557.0</td>
      <td>894.0</td>
      <td>132.0</td>
      <td>1138.0</td>
      <td>1145.0</td>
      <td>8466.0</td>
      <td>827.0</td>
      <td>426.0</td>
      <td>167.0</td>
      <td>896.0</td>
      <td>1400.0</td>
      <td>1935.0</td>
      <td>261.0</td>
      <td>1147.0</td>
      <td>248.0</td>
      <td>156.0</td>
      <td>278.0</td>
      <td>215.0</td>
      <td>1082.0</td>
      <td>368.0</td>
      <td>202.0</td>
      <td>363.0</td>
      <td>4135.0</td>
      <td>1132.0</td>
      <td>658.0</td>
      <td>379.0</td>
      <td>3191.0</td>
      <td>285.0</td>
      <td>691.0</td>
      <td>76.0</td>
      <td>237.0</td>
      <td>1638.0</td>
      <td>80.0</td>
      <td>1048.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>The Other 98%</th>
      <td>2.0</td>
      <td>0.0</td>
      <td>26726.0</td>
      <td>3899.0</td>
      <td>483.0</td>
      <td>688.0</td>
      <td>0.0</td>
      <td>4030.0</td>
      <td>33767.0</td>
      <td>1801.0</td>
      <td>5592.0</td>
      <td>693.0</td>
      <td>2855.0</td>
      <td>4.0</td>
      <td>3.0</td>
      <td>4338.0</td>
      <td>23015.0</td>
      <td>219688.0</td>
      <td>50881.0</td>
      <td>15792.0</td>
      <td>87643.0</td>
      <td>1579.0</td>
      <td>2502.0</td>
      <td>3997.0</td>
      <td>14073.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>9494.0</td>
      <td>3725.0</td>
      <td>3261.0</td>
      <td>11571.0</td>
      <td>3391.0</td>
      <td>2616.0</td>
      <td>4781.0</td>
      <td>11222.0</td>
      <td>5947.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1290.0</td>
      <td>67482.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>Politico</th>
      <td>47.0</td>
      <td>10.0</td>
      <td>25.0</td>
      <td>92.0</td>
      <td>163.0</td>
      <td>36.0</td>
      <td>6.0</td>
      <td>5.0</td>
      <td>4.0</td>
      <td>26.0</td>
      <td>27.0</td>
      <td>18.0</td>
      <td>268.0</td>
      <td>35.0</td>
      <td>18.0</td>
      <td>10.0</td>
      <td>85.0</td>
      <td>8.0</td>
      <td>138.0</td>
      <td>591.0</td>
      <td>154.0</td>
      <td>12.0</td>
      <td>13.0</td>
      <td>18.0</td>
      <td>10.0</td>
      <td>80.0</td>
      <td>24.0</td>
      <td>28.0</td>
      <td>243.0</td>
      <td>68.0</td>
      <td>5.0</td>
      <td>200.0</td>
      <td>205.0</td>
      <td>1.0</td>
      <td>260.0</td>
      <td>35.0</td>
      <td>446.0</td>
      <td>8.0</td>
      <td>41.0</td>
      <td>704.0</td>
      <td>...</td>
      <td>13.0</td>
      <td>82.0</td>
      <td>245.0</td>
      <td>62.0</td>
      <td>29.0</td>
      <td>10.0</td>
      <td>30.0</td>
      <td>12.0</td>
      <td>51.0</td>
      <td>5.0</td>
      <td>363.0</td>
      <td>101.0</td>
      <td>53.0</td>
      <td>29.0</td>
      <td>37.0</td>
      <td>7.0</td>
      <td>3.0</td>
      <td>90.0</td>
      <td>64.0</td>
      <td>222.0</td>
      <td>13.0</td>
      <td>7.0</td>
      <td>162.0</td>
      <td>20.0</td>
      <td>133.0</td>
      <td>13.0</td>
      <td>3.0</td>
      <td>6.0</td>
      <td>663.0</td>
      <td>113.0</td>
      <td>37.0</td>
      <td>94.0</td>
      <td>18.0</td>
      <td>7.0</td>
      <td>15.0</td>
      <td>16.0</td>
      <td>1286.0</td>
      <td>94.0</td>
      <td>1127.0</td>
      <td>190.0</td>
    </tr>
    <tr>
      <th>CNN Politics</th>
      <td>8.0</td>
      <td>6.0</td>
      <td>122.0</td>
      <td>19.0</td>
      <td>119.0</td>
      <td>21.0</td>
      <td>58.0</td>
      <td>29.0</td>
      <td>46.0</td>
      <td>33.0</td>
      <td>17.0</td>
      <td>75.0</td>
      <td>21.0</td>
      <td>101.0</td>
      <td>465.0</td>
      <td>19.0</td>
      <td>787.0</td>
      <td>78.0</td>
      <td>85.0</td>
      <td>170.0</td>
      <td>28.0</td>
      <td>279.0</td>
      <td>3.0</td>
      <td>187.0</td>
      <td>142.0</td>
      <td>87.0</td>
      <td>17.0</td>
      <td>55.0</td>
      <td>46.0</td>
      <td>130.0</td>
      <td>6.0</td>
      <td>72.0</td>
      <td>51.0</td>
      <td>156.0</td>
      <td>11.0</td>
      <td>119.0</td>
      <td>150.0</td>
      <td>17.0</td>
      <td>95.0</td>
      <td>101.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>ABC News Politics</th>
      <td>0.0</td>
      <td>1.0</td>
      <td>34.0</td>
      <td>35.0</td>
      <td>568.0</td>
      <td>23.0</td>
      <td>46.0</td>
      <td>7.0</td>
      <td>7.0</td>
      <td>152.0</td>
      <td>8.0</td>
      <td>0.0</td>
      <td>37.0</td>
      <td>4.0</td>
      <td>10.0</td>
      <td>10.0</td>
      <td>3.0</td>
      <td>0.0</td>
      <td>15.0</td>
      <td>56.0</td>
      <td>64.0</td>
      <td>6.0</td>
      <td>289.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>5.0</td>
      <td>30.0</td>
      <td>50.0</td>
      <td>0.0</td>
      <td>13.0</td>
      <td>25.0</td>
      <td>2.0</td>
      <td>25.0</td>
      <td>15.0</td>
      <td>3.0</td>
      <td>23.0</td>
      <td>5.0</td>
      <td>117.0</td>
      <td>6.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>Right Wing News</th>
      <td>57.0</td>
      <td>247.0</td>
      <td>182.0</td>
      <td>60.0</td>
      <td>29.0</td>
      <td>5434.0</td>
      <td>615.0</td>
      <td>401.0</td>
      <td>591.0</td>
      <td>179.0</td>
      <td>252.0</td>
      <td>193.0</td>
      <td>21.0</td>
      <td>33.0</td>
      <td>246.0</td>
      <td>248.0</td>
      <td>21.0</td>
      <td>2361.0</td>
      <td>203.0</td>
      <td>309.0</td>
      <td>272.0</td>
      <td>748.0</td>
      <td>1640.0</td>
      <td>52.0</td>
      <td>635.0</td>
      <td>33.0</td>
      <td>87.0</td>
      <td>244.0</td>
      <td>471.0</td>
      <td>1925.0</td>
      <td>645.0</td>
      <td>117.0</td>
      <td>52.0</td>
      <td>31.0</td>
      <td>441.0</td>
      <td>47.0</td>
      <td>51.0</td>
      <td>110.0</td>
      <td>266.0</td>
      <td>46.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>Eagle Rising</th>
      <td>593.0</td>
      <td>89.0</td>
      <td>1966.0</td>
      <td>155.0</td>
      <td>7577.0</td>
      <td>1193.0</td>
      <td>101.0</td>
      <td>22.0</td>
      <td>6.0</td>
      <td>29.0</td>
      <td>38.0</td>
      <td>26.0</td>
      <td>21.0</td>
      <td>1507.0</td>
      <td>17.0</td>
      <td>201.0</td>
      <td>12.0</td>
      <td>138.0</td>
      <td>264.0</td>
      <td>2.0</td>
      <td>6.0</td>
      <td>18.0</td>
      <td>2828.0</td>
      <td>210.0</td>
      <td>28.0</td>
      <td>54.0</td>
      <td>74.0</td>
      <td>500.0</td>
      <td>99.0</td>
      <td>266.0</td>
      <td>182.0</td>
      <td>170.0</td>
      <td>5.0</td>
      <td>58.0</td>
      <td>218.0</td>
      <td>36.0</td>
      <td>3304.0</td>
      <td>40.0</td>
      <td>14.0</td>
      <td>75.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>Freedom Daily</th>
      <td>1511.0</td>
      <td>572.0</td>
      <td>65.0</td>
      <td>947.0</td>
      <td>2965.0</td>
      <td>13305.0</td>
      <td>728.0</td>
      <td>3442.0</td>
      <td>3700.0</td>
      <td>1860.0</td>
      <td>4595.0</td>
      <td>1203.0</td>
      <td>246.0</td>
      <td>95.0</td>
      <td>197.0</td>
      <td>734.0</td>
      <td>54.0</td>
      <td>82.0</td>
      <td>981.0</td>
      <td>1747.0</td>
      <td>51.0</td>
      <td>130.0</td>
      <td>71.0</td>
      <td>2382.0</td>
      <td>106.0</td>
      <td>1279.0</td>
      <td>3697.0</td>
      <td>18054.0</td>
      <td>334.0</td>
      <td>182.0</td>
      <td>104.0</td>
      <td>2889.0</td>
      <td>6053.0</td>
      <td>1791.0</td>
      <td>32.0</td>
      <td>137.0</td>
      <td>3304.0</td>
      <td>1474.0</td>
      <td>319.0</td>
      <td>192.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>9 rows × 536 columns</p>
</div>




```
subset(shareDF, not.isna())
```


      File "<ipython-input-62-643c2f0e51b7>", line 1
        subset(shareDF, not.isna())
                           ^
    SyntaxError: invalid syntax




```
fig, ax = plt.subplots()
ax.boxplot(shareDF.dropna(axis=1), showfliers=True, showmeans=Trued)
ax.set_xticklabels(pages)
plt.xticks(rotation='vertical')
plt.yscale('log')
plt.show()
```


![png](DataStorytellingWriteUp_files/DataStorytellingWriteUp_24_0.png)



```
meansDF
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
      <th>AvgReaction</th>
      <th>AvgComment</th>
      <th>AvgShare</th>
    </tr>
    <tr>
      <th>Page</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Occupy Democrats</th>
      <td>34668.736842</td>
      <td>2858.100478</td>
      <td>28086.837321</td>
    </tr>
    <tr>
      <th>Addicting Info</th>
      <td>3120.307143</td>
      <td>392.264286</td>
      <td>1215.914286</td>
    </tr>
    <tr>
      <th>The Other 98%</th>
      <td>20799.106557</td>
      <td>907.163934</td>
      <td>15645.655738</td>
    </tr>
    <tr>
      <th>Politico</th>
      <td>900.296642</td>
      <td>170.231343</td>
      <td>180.998134</td>
    </tr>
    <tr>
      <th>CNN Politics</th>
      <td>677.657702</td>
      <td>322.097800</td>
      <td>182.029340</td>
    </tr>
    <tr>
      <th>ABC News Politics</th>
      <td>176.670000</td>
      <td>71.435000</td>
      <td>39.140000</td>
    </tr>
    <tr>
      <th>Right Wing News</th>
      <td>2453.917910</td>
      <td>359.988806</td>
      <td>1393.167910</td>
    </tr>
    <tr>
      <th>Eagle Rising</th>
      <td>519.989510</td>
      <td>79.482517</td>
      <td>596.734266</td>
    </tr>
    <tr>
      <th>Freedom Daily</th>
      <td>3652.205357</td>
      <td>511.616071</td>
      <td>2452.294643</td>
    </tr>
  </tbody>
</table>
</div>




```
meansDF
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
      <th>AvgReaction</th>
      <th>AvgComment</th>
      <th>AvgShare</th>
    </tr>
    <tr>
      <th>Page</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Occupy Democrats</th>
      <td>34668.736842</td>
      <td>2858.100478</td>
      <td>28086.837321</td>
    </tr>
    <tr>
      <th>Addicting Info</th>
      <td>3120.307143</td>
      <td>392.264286</td>
      <td>1215.914286</td>
    </tr>
    <tr>
      <th>The Other 98%</th>
      <td>20799.106557</td>
      <td>907.163934</td>
      <td>15645.655738</td>
    </tr>
    <tr>
      <th>Politico</th>
      <td>900.296642</td>
      <td>170.231343</td>
      <td>180.998134</td>
    </tr>
    <tr>
      <th>CNN Politics</th>
      <td>677.657702</td>
      <td>322.097800</td>
      <td>182.029340</td>
    </tr>
    <tr>
      <th>ABC News Politics</th>
      <td>176.670000</td>
      <td>71.435000</td>
      <td>39.140000</td>
    </tr>
    <tr>
      <th>Right Wing News</th>
      <td>2453.917910</td>
      <td>359.988806</td>
      <td>1393.167910</td>
    </tr>
    <tr>
      <th>Eagle Rising</th>
      <td>519.989510</td>
      <td>79.482517</td>
      <td>596.734266</td>
    </tr>
    <tr>
      <th>Freedom Daily</th>
      <td>3652.205357</td>
      <td>511.616071</td>
      <td>2452.294643</td>
    </tr>
  </tbody>
</table>
</div>




