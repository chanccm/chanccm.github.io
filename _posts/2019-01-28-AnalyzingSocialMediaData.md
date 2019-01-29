---
layout: post
title: "Analyzing Social Media Data"
date: 2019-01-28
categories: [Python, datascience, social media, web scraping]
---

![SocialMediaYouTubeData](/assets/images/SocialMediaYouTubeData.png){:class="img-responsive" width="600"}
<br/>

**What do people watch on Youtube?** I got curious as I learn how to analyze social media data. So I go to [SocialBlade](https://socialblade.com), look at the "Top 500 YouTube Influential YouTube Channels (sorted by SB rank)", and apply the lxml parser to generate the above data. In other words, I need to extract data from a website by web scraping, followed by cleaning up and organizing the data into a format (dataframe) that I can use.

First, to get a table of the name of YouTube channels, number of views and subscribers

{% highlight python %}

import lxml.html as lh

url = 'https://socialblade.com/youtube/top/500'
r = requests.get(url)
doc = lh.fromstring(r.content)

# Need to find \<div> elements with an attribute that contains the specific string that will lead to the data that I need: in this case, text of a certain size and background color
table_data = doc.xpath("//div[contains(@style, 'width: 860px; background: #f')]")
# Data will be collected as a list of HTML element.

# To convert the data into a readable list and name it 'col':

col = []

for t in table_data:
    name = t.text_content()
    col.append((name))

{% endhighlight %}

The first entry of the list is something like: 

13th

A+ 

Toys and Colors


199

8,352,442 

4,637,771,013 

Next, I convert the list 'col' into a dataframe:

{% highlight python %}

import pandas as pd

df = pd.DataFrame(col)

{% endhighlight %}
<br/>
                                                  0
0   \n1st\n\nA++ \n\n\nT-Series\n\n\n13,062\n\n82,...

The third step is to The information extract the information contained in that one long string of text. Need to ignore the unimportant parts , and split them up into individual columns of data:

{% highlight python %}
df1 = df[0].str.split('\n', expand=True)

# (Need to further clean up the dataframe, because the original table on the website is misaligned. Some entries have more columns than others. I will skip those steps here)

# Give proper names to each column:
df1.columns = ['Rank', 'Grade', 'Username', 'Uploads', 'Subs', 'Video Views']

# Convert str into numbers for the 'Uploads', 'Subs', and 'Video Views' columns
df1['Uploads'] = df1['Uploads'].str.replace(',', '')
df1['Subs'] = df1['Subs'].str.replace(',', '')
df1['Video Views'] = df1['Video Views'].str.replace(',', '')

df1['Uploads'] = pd.to_numeric(df1['Uploads'])
df1['Subs'] = pd.to_numeric(df1['Subs'])
df1['Video Views'] = pd.to_numeric(df1['Video Views'])

{% endhighlight %}

 To find out which YouTube channel belongs to which category, I use a different method (etree.tostring) to extract that information than lh.fromstring used previously.

{% highlight python %}
from lxml import etree

table_data_list = []

for j in table_data:
    data_for_list = etree.tostring(j)
    table_data_list.append(data_for_list)

# To extract categorical data for each entry and put it into a list:
table_data_list_category = []

for k in table_data_list:
    data_for_category = k.decode("utf-8")
    data_for_category = data_for_category.split('Category: ', 1)[1].split('"', 1)[0]
    table_data_list_category.append(data_for_category)   
    
# Add the catagorical data to the dataframe generated above:
df1['Category'] = table_data_list_category
    
{% endhighlight %} 

Importantly, next find out the sum of subscription and views for each category after grouping YouTube channels of the same category together:

{% highlight python %}
Num_Subs_by_Category = df1.Subs.groupby(df1['Category']).sum().sort_values(ascending= False)/1000000


Num_Views_by_Category = df1['Video Views'].groupby(df1['Category']).sum().sort_values(ascending= False)/1000000

# Note: the counts are in millions

{% endhighlight %} 

We are almost there! Finally I plot the data in a barchart:

{% highlight python %}

import seaborn as sns
import matplotlib.pyplot as plt

f, axes = plt.subplots(1, 2, figsize=(15,10))
sns.set_context("poster")
g1 = sns.barplot(x= 'Subs', y = 'Category', data = Num_Subs_by_Category_df, ax=axes[0])
g1.set(xlabel='Total number of subscriptions (x million)')
g2 = sns.barplot(x= 'Video Views', y = 'Category', data = Num_Views_by_Category_df, ax=axes[1])
g2.set(xlabel='Total number of views (x million)')
plt.tight_layout()
plt.show()

{% endhighlight %} 

Looks like music and entertainment have the most subscribers and views. Next to these, people watch games and look up information about file(movies) on YouTube. Education (including shows designed for toddlers, young children)- related channels are pretty high up as well. Contrary to popular believe about people are watching cute cats all day on YouTube, the 'animal' class falls outside of the top 10 most popular categories.
