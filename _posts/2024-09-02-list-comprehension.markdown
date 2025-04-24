---
title: List Comprehension"
tags: [Python]
author: Sangeun
---

## List Comprehension

For the high quality and high efficiency, We can use a _List Comprehension_. It allows to write more readable code when creating lists.

### Structure
```python
list_variable = [ expression for item in iterable if condition]
```
**Here is the example.**
```python
animals = ['cat', 'dog', 'lion', 'rabbit', 'bee', 'fly']
animal_example = ['cat', 'dog', 'lion', 'rabbit']
only_insects = [animal for animal in animals if animal not in animal_example]

print(only_insects)
```

If run this code, the result will be bee and fly. This code extract items if it's not in the animal_example.
