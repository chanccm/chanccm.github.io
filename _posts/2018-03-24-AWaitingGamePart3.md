---
layout: post
title: "A Waiting Game Part 3"
date: 2018-03-24
categories: [datascience, research, data-visualization]
---

Sometimes in trying to obtain the most useful set of features for predicting an outcome may require us to define new features based on the existing ones. I learnt this from the ['Pytanic' tutorial on kaggle, by Heads or Tails.](https://www.kaggle.com/headsortails/pytanic)

Hence I want to define a list of **derived or engineered features**:
'AverageWaitTime' - as in on average how many minutes one is willing to wait for someone regardless of whether that person is his/her superior, family, friend, or lover (one averaged value by averaging the 4 categories)
<br/>
'AverageMakeOthersWaitTime' -  as in on average one is comfortable making other people wait.
<br/>
'WaitSuperior%Difference' - (WaitforSuperior - MakeSuperiorWait)/WaitforSuperior
<br/>
'WaitFamily%Difference' - (WaitforFamily - MakeFamilyWait)/WaitforFamily
<br/>
'WaitFriend%Difference' - (WaitforFriend - MakeFriendWait)/WaitforFriend
<br/>
'WaitLover%Difference' - (WaitforLover - MakeLoverWait)/WaitforLover
<br/>

Using Lasso regression again to see which features matter:

![JpHWWaittimeLassoisJp3.png](/assets/images/JpHWWaittimeLassoisJp3.png){:class="img-responsive"}

![JpHWWaittimeLassoGender3.png](/assets/images/JpHWWaittimeLassoGender3.png){:class="img-responsive"}

To predict whether the unknown subject is Japanese, only the derived features, 'WaitFamily%Difference' and 'WaitLover%Difference' seem to affect the prediction results.

As for predicting whether the subject is male or female, only 'WaitLover%Difference' should be included in the set of features.

With these in mind, I computed cross validation accuracy scores again,

{% highlight python %}
x_isJ = df2.loc[:,['WaitforFriend','MakeFamilyWait','MakeFriendWait','MakeLoverWait','WaitFamily%Difference','WaitLover%Difference']]
y_isJ = df2['isJapanese']
x_G = df2.loc[:,['WaitforSuperior','WaitforFriend','MakeSuperiorWait','MakeFamilyWait','MakeFriendWait','MakeLoverWait','WaitLover%Difference']]
y_G = df2['GenderScore']
knn = KNeighborsClassifier(n_neighbors=3)
isJ_cv_scores = cross_val_score(knn, x_isJ, y_isJ, cv=4)
print('Predicting whether Japanese: ', isJ_cv_scores)
print('Mean: ', isJ_cv_scores.mean())

knn = KNeighborsClassifier(n_neighbors=3)
G_cv_scores = cross_val_score(knn, x_G, y_G, cv=4)
print('Predicting gender: ', G_cv_scores)
print('Mean: ', G_cv_scores.mean())
{% endhighlight %}

Predicting whether Japanese:  [ 0.88888889  0.88888889  0.875       0.5       ]
<br/>
Mean:  0.788194444444
<br/>
Predicting gender:  [ 0.77777778  0.77777778  0.625       0.625     ]
<br/>
Mean:  0.701388888889

With derived features, there is a slight improvement for predicting the natonality, while not much help for predicting gender.

Finally, let me introduce another useful modeling method for categorial dependent variable: the **logistic regression**.

{% highlight python %}
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import cross_val_score

x_isJ = df2.loc[:,['WaitforFriend','MakeFamilyWait','MakeFriendWait','MakeLoverWait','WaitFamily%Difference','WaitLover%Difference']]
y_isJ = df2['isJapanese']
x_G = df2.loc[:,['WaitforSuperior','WaitforFriend','MakeSuperiorWait','MakeFamilyWait','MakeFriendWait','MakeLoverWait','WaitLover%Difference']]
y_G = df2['GenderScore']
logreg =  LogisticRegression()
isJ_cv_scores = cross_val_score(logreg, x_isJ, y_isJ, cv=4)
print('Predicting whether Japanese: ', isJ_cv_scores)
print('Mean: ', isJ_cv_scores.mean())

G_cv_scores = cross_val_score(logreg, x_G, y_G, cv=4)
print('Predicting gender: ', G_cv_scores)
print('Mean: ', G_cv_scores.mean())
{% endhighlight %}

Predicting whether Japanese:  [ 0.77777778  0.66666667  0.625       1.        ]
<br/>
Mean:  0.767361111111
<br/>
Predicting gender:  [ 0.88888889  0.77777778  0.75        0.5       ]
<br/>
Mean:  0.729166666667

Using logistic regression, the accuracy score for predicting gender improved a little.

To summarize the cross validation accuracies using knn and logistic regression:

|      | knn (n_neighbors = 8) | knn (n_neighbors = 3)| knn with selected features only | knn with selected and derived features | logistic regression |
|----------|:----------:|:----------:|:----------:|:----------:|:----------:|
| Nationality cv accuracy: | 0.61 | 0.67 | 0.76 | 0.79 | 0.77 |
| Gender cv accuracy:   | 0.64 | 0.64 | 0.70 | 0.70 | 0.73 |

