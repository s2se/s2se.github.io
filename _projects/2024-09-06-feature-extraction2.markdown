---
title: Feature Extraction- 2. WIKI
description: Python, postgreSQL
date:   2024-08-21 21:03:36 +0900

---

## Description  
## Feature Extraction2 

### The problem   
1. A consequence of a consistent strive to innovate on a large, multi-national scale, which results in a massive amount of data.​  

2. At this scale, manually finding the key information (features) in an international haystack of data is infeasible. ​ 

3. To maintain their reputation as an innovation leader, Ericsson cannot afford to waste time on these outdated, manual approaches for feature extraction.  

### Approach 1
1. Web crawling: Get the text from Whirlpool website. 

```python
import requests
import bs4


URL = "https://forums.whirlpool.net.au/thread/3n4wj8v3"
response = requests.get(URL)
soup = bs4.BeautifulSoup(response.text,"html.parser")
```

However, Whirlpool does not allow to access their source code.  
<img width="1440" alt="Sourcecode_whirlpool" src="https://github.com/user-attachments/assets/efb4007d-777b-4319-ae87-8887372dd209">  

Therefore, instead of Whirlpool, I will use Wikipedia. I've decided to use the page for Sydney Observatory, which is my favorite place in Sydney.

```python
import wikipediaapi

wiki = wikipediaapi.Wikipedia(language='en', user_agent='Example/1.0')
page_sydob = wiki.page('Sydney Observatory')
print("Page - Exists: %s" % page_sydob.exists())


extract = wikipediaapi.ExtractFormat.WIKI
print("Page - Full Text: %s" % page_sydob.text)
```
**After run this code, if it is successfully work, you can get this result.**  
:Page - Exists: True  
Page - Full Text: The Sydney Observatory is a heritage-listed meteorological station, astronomical observatory, function venue, science museum, and education facility located on Observatory Hill at Upper Fort Street, in the inner city Sydney suburb of Millers Point in the City of Sydney local government area of New South Wales, Australia. It was designed by William Weaver (plans) and Alexander Dawson (supervision) and built from 1857 to 1859 by Charles Bingemann & Ebenezer Dewar. It is also known as The Sydney Observatory; Observatory; Fort Phillip; Windmill Hill; and Flagstaff Hill. It was added to the New South Wales State Heritage Register on 22 December 2000.
The site was formerly a defence fort, semaphore station, time ball station, meteorological station, observatory and windmills. The site evolved from a fort built on 'Windmill Hill' in the early 19th century to an observatory within the following 100 years. It is now a working museum where evening visitors can observe the stars and planets through a modern 40-centimetre (16 in) Schmidt-Cassegrain telescope and an historic 29-centimetre (11 in) refractor telescope built in 1874, the oldest telescope in Australia in regular use. ...

```python
import wikipediaapi

wiki = wikipediaapi.Wikipedia(user_agent='Example/1.0',language='en')
page_sydob = wiki.page('Sydney Observatory')

with open("sydney_observatory.txt", "w") as f:
    f.write(page_sydob.text)
```

**After run this code, you can find the "sydney_observatory.txt" file inside of your folder.**  
<img width="648" alt="Screenshot 2024-10-02 at 2 37 47 PM" src="https://github.com/user-attachments/assets/20915c3e-d87d-4102-8a80-cd2f87684b26">  

### Now, Let's bring the text file
```python
from nltk.tokenize import word_tokenize

f= open("sydney_observatory.txt", "r")
data= f.read().split()
print(data)

f.close()
```

The result will be: 
['The',
 'Sydney',
 'Observatory',
 'is',
 'a',
 'heritage-listed',
 'meteorological',
 'station,',
 'astronomical',
 'observatory,',
 'function', ]

### Deal with Stopwords

```python
 from nltk.corpus import stopwords
 stop_words = set(stopwords.words('english'))

cleaned_data = []
for word in data:
    if word not in stop_words:
        cleaned_data.append(word)
        
print(cleaned_data)
```  
 ### Let's find the Keywords from this text

1. Before counting the keywords from the text, we want to complete stemming. There are a few algorithm options we have:
    1) Porter Algorithm
    2) Lancaster Stemmer 
2. Stemming with two algorithms
3. Comparing how the results differ


```python
from nltk.stem import PorterStemmer
from nltk.stem import LancasterStemmer

porter_stemmer = PorterStemmer()
lancaster_stemmer = LancasterStemmer()

cleaned_by_porter = []
cleaned_by_lancaster = []

for word in cleaned_data:
    cleaned_by_porter.append(porter_stemmer.stem(word))

for word in cleaned_data:
    cleaned_by_lancaster.append(lancaster_stemmer.stem(word))
```

### Let's Count it!

```python
from collections import Counter
result_counts_1 = Counter(cleaned_by_porter)
print(result_counts_1)

result_counts_2 = Counter(cleaned_by_lancaster)
print(result_counts_2)
```

- The result of porter:  
Counter({'observatori': 86, 'the': 64, 'sydney': 43, 'new': 30, 'time': 30, 'astronom': 29, '–': 27, 'site': 26, 'in': 24, 'fort': 23, 'govern': 23, 'work': 19, 'build': 18, 'first': 17, 'signal': 17
- The result of lancaster
Counter({'observ': 95, 'the': 65, 'sydney': 43, 'astronom': 37, 'govern': 30, 'new': 30, 'tim': 30, '–': 27, 'sit': 26, 'in': 25, 'fort': 23, 'stat': 20, 'work': 19, 'sign': 18, 'build': 18, 'first': 17
