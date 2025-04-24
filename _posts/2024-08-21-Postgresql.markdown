---
title: Link with Python
tags:  [Postgresql] 
author: Sangeun

---

## Link between PostgreSql and Python

1. Needs to install PostgreSql library in Python

```python
!pip install psycopg2-binary
```
If you install that in your mac, they initialize user name, password automatically.
You can check that on the website.
[Here](https://postgresapp.com/)  
<img width="535" alt="Mac_pass" src="https://github.com/user-attachments/assets/d564379d-f695-45de-be57-d5e2f7304702">

**Now Let's connect postgreSql into python**
```python
connection = psycopg2.connect("host='localhost' dbname=yourDBname user=Yourmacusername password=none port=5432")
```
Build cursor for deal with each row.
```python
cur = connection.cursor()
```
Make a new table.
```python
cur.execute("CREATE TABLE test_table (title varchar, content text);") 
```
