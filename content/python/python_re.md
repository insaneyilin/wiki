---
title: "Python 正则表达式"
date: 2019-11-20 11:01
tag: python
---

正则表达式(regular expression)是一种用来匹配字符串的强有力的武器。它的设计思想是用一种描述性的语言来给字符串定义一个规则，凡是符合规则的字符串，我们就认为它“匹配”了，否则，该字符串就是不合法的。

Python 中可以用 `re` 模块使用正则表达式。

```python
import re

# 由于Python的字符串本身也用\转义，所以要特别注意
s = 'ABC\\-001' # Python的字符串
# 对应的正则表达式字符串变成：
# 'ABC\-001'

# 强烈建议使用Python的r前缀，就不用考虑转义的问题了
s = r'ABC\-001' # Python的字符串
# 对应的正则表达式字符串不变：
# 'ABC\-001'

if re.match(r'^\d{3}\-\d{3,8}$', '010-12345'):
    print("match success")
# match 成功会返回一个 Match 对象，否则返回 None

# 使用预编译提高效率
re_telephone = re.compile(r'^(\d{3})-(\d{3,8})$')
re_telephone.match('010-12345').groups()
# ('010', '12345')
```