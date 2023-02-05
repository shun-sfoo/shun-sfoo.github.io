# 基本数据类型

## 整形

char,
short (至少 16 位),
int(至少和 short 一样长),
long (至少 32 位， 且至少和 int 一样长),
long long(至少 64 位， 且至少和 long 一样长)
每种类型都有有符号和无符号两种

C++11 初始化方式

```cpp
int n = {23};
int emus{7};
int rocs = {}; // 0
int psychics = {} // 0
```

## C++11 auto 声明

让编译器能根据初始值的类型推断变量的类型

```cpp
auto n = 100;
auto x = 1.5;
auto y = 1.3e12L;
std::vector<double> scores;
auto pv = scores.begin();
```
