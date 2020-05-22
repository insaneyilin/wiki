---
title: "Online Judge 解题小技巧"
date: 2019-07-12 01:58
collection: algorithm interviews
---

记录一些小 tips 。

## 输入输出

输入数据，以 `EOF` 结束：

```cpp
while (scanf("%d", &n) != EOF) {
  // ...
}
```

输入数据，以 0 结束：

```cpp
while (scanf("%d", &n) != EOF && n != 0) {
  // ...
}
```

输入数据，有多个 cases ，先输入 cases 数目：

```cpp
scanf("%d", &num_cases);
for (int i = 0 i < num_cases; ++i) {
  //...
}
```

整行输入：

```cpp
// C语言风格
char buff[256];
gets(buff); // 会接受空格、制表符('\t')以及回车键('\n')

// C++ 风格
string str;
getline(cin, str); // 注意 getline() 不会接受换行符('\n')
```

重定向输入输出：

```cpp
#define LOCAL
#include <stdio.h>
#define INF 1000000000

int main() {
#ifdef LOCAL
  freopen("data.in", "r", stdin);   //重定向输入
  freopen("data.out", "w", stdout); //重定向输出
#endif
  /* 之后的一切标准输入输出都会变为文件输入输出,
  如scanf,printf实际是从文件输入输出,对cin与cout也适用 */
  return 0;
}
```

```cpp
// C++ 风格从文件中进行输入输出
#include <fstream>

/* iftream 与 ofstream */
ifsream fin("data.in");
ofsream fout("data.out");
```
