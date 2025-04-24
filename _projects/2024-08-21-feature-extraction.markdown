---
title: Feature Extraction- 1. Email
description: Python, PostgreSQL
date:   2024-08-21 21:03:36 +0900

---

## Description  
## Feature Extraction1
### The problem   
1. A consequence of a consistent strive to innovate on a large, multi-national scale, which results in a massive amount of data.​  

2. At this scale, manually finding the key information (features) in an international haystack of data is infeasible. ​ 

3. To maintain their reputation as an innovation leader, Ericsson cannot afford to waste time on these outdated, manual approaches for feature extraction.  

## Approach1 : Understanding the data to be used  

| Email                                                    | whirlpool Forum                                                                    |
|----------------------------------------------------------|------------------------------------------------------------------------------------|
| It is semi-structured, and the content is unpredictable. | Forum where customers discuss customer services.  Similarly, there is no structure. |  


## Approach2 : Identifying the tools to be used.  
1. imap_tools
2. Spacy
3. postgresql  

First of all, make the **account.py** file. Should be your email account.  
Password should be app password.

```python
Email_address = "youremail@gmail.com"
Email_password = "password"
```
Save each email into the execl file.

```python
from imap_tools import MailBox
from account import *
import pandas as pd

mailbox =MailBox("imap.gmail.com", 993)
mailbox.login(Email_address, Email_password, initial_folder= "INBOX")
sample_email = pd.DataFrame(columns =['title', 'send', 'get', 'date', 'message'])

for msg in mailbox.fetch(limit=20, reverse=True):
    date =  msg.date
    date2 =date.strftime('%Y-%m-%d')
    title = msg.subject
    send = msg.from_
    get =  msg.to
    message = msg.text
    raw_data = {'title' : title,
                'send' : send,
                'get' :  get,
                'date' :  date2,
                'message' : message
                }

    sample_email= sample_email.append(raw_data, ignore_index =True)
  
sample_email.to_excel('sample.xlsx', encoding = 'utf-8')
mailbox.logout()

```
<img width="550" alt="Screenshot 2024-09-01 at 4 46 45 PM" src="https://github.com/user-attachments/assets/b88d4003-26dd-40ab-bca9-d9286f4d8c35">  
We got a 20 rows and 5 of each columns. 

Open email_extraction.py file.

```python
import pandas as pd
import spacy
import re 

df = pd.read_excel('sample.xlsx')
if df['message'].isnull().any():
    df['message'].fillna('', inplace=True)
    
nlp = spacy.load("en_core_web_sm")
df['body'] = df['message'].astype(str)
df['body'] = df['body'].apply(nlp)

tokenized_sentence = [
    [re.sub(r"[^가-힣a-zA-Z0-9]", "", token.text.lower())
    for token in doc
    if not token.is_stop]
    for doc in df['body']
    ]

fianl_tokenized_sentence = [
    [token for token in tokens if token]
    for tokens in tokenized_sentence
]

print(fianl_tokenized_sentence)

```
#### But this model can not deal with Korean sentences.
For solve this, need to set Korean Spacy model.

Write down this code on your terminal.  
**_"_**  
### _!python -m spacy download ko_core_news_sm_
**_"_**  


```python
import pandas as pd
import spacy

df = pd.read_excel('sample.xlsx')
nlp2= spacy.load("ko_core_news_sm")
df['body'] = df['message'].astype(str)
df['body'] = df['body'].apply(nlp2)

korean_sentence =[ [token.text for token in doc if token is not token.is_stop and token.tag_ == 'ncn' and token.text.isalpha()]
    for doc in df['body']
]
print(korean_sentence)
```
Given Result is
<img width="738" alt="Result_after_spacy" src="https://github.com/user-attachments/assets/b67733a7-f85c-4349-917f-d1d613cf19b8">

**Let's count the number of words**
```python
from collections import Counter
korean_words =[ [token.text for token in doc if token is not token.is_stop and token.tag_ == 'ncn' and token.text.isalpha()]
    for doc in df['body']]

count_words =[words for sublist in korean_words for words in sublist ]
count_ =Counter(count_words)
print(count_)
```
<img width="739" alt="After_count" src="https://github.com/user-attachments/assets/872eb766-5162-4f4f-a192-0f7cea507c98">

**Now save the dictionaries into PostgreSql**  
Make the database
```sql
create database email_extraction;
```
Connect with Python
```python
import psycopg2
connection = psycopg2.connect("host='localhost' dbname=email_extraction user=yourusername password=yourpassword port=5432")
cur = connection.cursor()
```
Create table
```python
cur.execute("CREATE TABLE Number_for_words_email ( word TEXT PRIMARY KEY, count INTEGER);") 
```
Put the items into Number_for_words_email table from the list 'count_'
```python
for word, count in count_.items():
    cur.execute(
        "INSERT INTO Number_for_words_email (word, count) VALUES (%s, %s) ON CONFLICT (word) DO UPDATE SET count = EXCLUDED.count;",
        (word, count)
    )
```
Check the the table
```python
cur.execute("SELECT * FROM Number_for_words_email;")
rows = cur.fetchall()

for row in rows:
    print(row)

connection.commit()
cur.close()
connection.close()
```
Given result; Here is the Some of the results.  
<img width="104" alt="Result" src="https://github.com/user-attachments/assets/19bb3bf9-4d44-4f5e-874f-b2de964beaa0">