---
title:  Solving encoding Error
tags:  [python] 
author: Sangeun
---

## Error
An attempt was made to load an Excel file, but if failed.
**_Error message Here_ unicodedecodeerror: 'utf-8' codec can't decode byte 0x84 in position 11:**
This error occurs when Python tries to decode the file using UTF-8 encoding, but some of them do not match the UTF-8.  
## Solving
To solve this issue, saving the file with Encoding as UTF-8.
```python
file_1.to_excel('example.xlsx', encoding = 'utf-8')

```

Now works. Can read file.