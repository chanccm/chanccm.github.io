---
layout: post
title: "Fun with WordCloud"
date: 2018-07-21
categories: [Python, datascience, data-visualization]
---

![CarmenWorldCloudResultBig](/assets/images/CarmenWorldCloudResultBig.png){:class="img-responsive" width="600"}
<br/>

I've been busy learning about Deep Learning. In the meantime, I found something fun and relatively easy to do: using the word cloud generator in Python. The code to create the above image is rather simple, and I'm using a modified version of Andreas Mueller's [Python word_cloud package](https://github.com/amueller/word_cloud). The text file is created from the text on [my LinkedIn page](https://www.linkedin.com/in/carmen-chan-ccm/)
<br/>

{% highlight python %}
from os import path
from wordcloud import WordCloud

d = path.dirname('_file_path_')

# Read the whole text.
text = open(path.join(d, 'text_file.txt')).read()

# Generate a word cloud image
wordcloud = WordCloud().generate(text)

# lower max_font_size
wordcloud = WordCloud(width=1000, height=500, max_font_size=120).generate(text) 

# write the image to a file
wordcloud.to_file(path.join(d, 'output_image_file.png'))

# show the image within the jupyter notebook window
plt.figure(figsize=(10,5))
plt.imshow(wordcloud, interpolation="bilinear")
plt.axis("off")
plt.show()
{% endhighlight %}
<br/>

Python wordcloud also allows something fancy, which is letting the user specify a mask to contain the distribution of words. Again I'm using a modified version of Mueller's code designed for ['Alice in Wonderland'](https://github.com/amueller/word_cloud/blob/master/examples/masked.py)
<br/>

{% highlight python %}
from os import path
from PIL import Image
import numpy as np
import matplotlib.pyplot as plt

from wordcloud import WordCloud, STOPWORDS

d = path.dirname('_file_path_')

# Read the whole text.
text = open(path.join(d, 'text_file.txt')).read()

horse_mask = np.array(Image.open(path.join(d, 'Horse_picture.png')))

stopwords = set(STOPWORDS)
stopwords.add("said")

wc = WordCloud(background_color="white",  mask=horse_mask,
               stopwords=stopwords) 

# generate word cloud
wc.generate(text)

# store to file
#wc.to_file(path.join(d, 'output_image_file.png'))

# show
plt.imshow(wc, interpolation='bilinear')
plt.axis("off")
#plt.figure(figsize=(20,20))
plt.axis("off")
plt.show()
{% endhighlight %}
<br/>

![CarmenWorldCloudResultHorse](/assets/images/CarmenWorldCloudResultHorse.png){:class="img-responsive" width="600"}