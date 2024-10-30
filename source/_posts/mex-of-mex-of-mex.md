---
title: 题解 - mex_of_mex_of_mex
categories:
  - Competitive Programming
  - 算法
tags:
  - mex
abbrlink: 32f51c88
date: 2024-10-24 17:48:47
---

# 题解 - mex of mex of mex

## 题意

[传送门](https://www.luogu.com.cn/problem/U495505)

给定一个数组，求其的所有子区间的mex构成的二维数组的**部分**子区间的mex构成的数组的mex。

## 关于

确实，题目很大程度上改编自 [中位数的中位数的中位数](https://qoj.ac/contest/1794/problem/9314)。主要是要出二分相关的，而这题确实给我印象很深刻，于是一琢磨，诶嘿。

其实想法首先来自那道 [Top Cluster](https://qoj.ac/problem/8235) 的某篇题解，其中提到了 mex 的常用技巧为二分，于是就想着 mex 可以套在哪里，然后就想到了那道中位数题。~~（顺便这题我现在还没写出来）~~

不过有了 idea 之后具体思路我想了好久...

很惭愧，本人还没有想到不预先计算 $b$ 数组，直接在外部套二分的类似那道中位数题的做法。

## 做法

~~你怎么这么多废话~~

因此首先计算 $b$ 数组，~~这应该是 trival 的~~，此处可以维护权值树状数组上某个值最后出现的位置，外面套个二分，也可以直接在权值线段树上二分，同样也维护某个值最后出现的位置，前者是 $O(n^2log^{2}n)$，后者是 $O(n^{2}logn)$ 的，两种都可以。

从 $b$ 数组计算 $c$ 数组的难点大概在于 4e6 个区间 mex 询问？~~实际上对拍的时候发现就算 $c$ 数组算错了答案也不一定错~~。

考虑一维问题我们是怎么做的，找到某个值最靠右的位置，以使其尽可能的发生贡献，如果直接套到二维，那就是考虑最靠 **右下** 的位置，使其尽可能对当前查询的区间有贡献。但直接维护应该是困难的。

于是我们观察到将 $b$ 数组二维展开，由于查询区间的主对角线和 $b$ 的主对角线重合，因此最靠 **下** 的位置也就是最靠 **右** 的，因此只需要维护每个数的 $i$ 轴坐标即可。

于是就做完了，从 $c$ 得出答案直接暴力即可。

## 代码

```cpp
#include <bits/stdc++.h>
#define lc p << 1
#define rc p << 1 | 1
#define mid (s + t) / 2

constexpr int N = 2e3 + 10;

int d[N << 2];

void update(int s, int t, int p, int x, int v, bool f) {
  if (s == t) {
    if (f) {
      d[p] = v;
    } else {
      d[p] = std::max(d[p], v);
    }
    return;
  }
  if (x <= mid) update(s, mid, lc, x, v, f);
  else update(mid + 1, t, rc, x, v, f);
  d[p] = std::min(d[lc], d[rc]);
}

int query(int s, int t, int p, int v) {
  if (s == t) return s;
  if (d[lc] < v) return query(s, mid, lc, v);
  return query(mid + 1, t, rc, v);
}

int main() {
  std::cin.tie(nullptr)->sync_with_stdio(false);
  int n;
  std::cin >> n;
  std::vector<int> a(n + 1);
  for (int i = 1; i <= n; i++) {
    std::cin >> a[i];
  }
  std::vector<std::vector<int>> b(n + 1, std::vector<int>(n + 1));

  for (int r = 1; r <= n; r++) {
    update(0, n + 2, 1, a[r], r, true);
    for (int l = 1; l <= r; l++) {
      b[l][r] = query(0, n + 2, 1, l);
    }
  }
  for (int i = 1; i <= n; i++) {
    for (int j = i; j <= n; j++) {
      update(0, n + 2, 1, b[i][j], 0, true);
    }
  }
  std::fill(d, d + (N << 2), 0);
  std::vector<std::vector<int>> c(n + 1, std::vector<int>(n + 1));
  std::vector<int> able(n + 3);
  for (int j = 1; j <= n; j++) {
    for (int i = 1; i <= j; i++) {
      update(0, n + 2, 1, b[i][j], i, false);
      if (i == j) {
        for (int l = 1; l <= j; l++) {
          c[l][j] = query(0, n + 2, 1, l);
          able[c[l][j]] = 1;
        }
      }
    }
  }
  for (int i = 0; i <= n + 2; i++) {
    if (!able[i]) {
      std::cout << i << '\n';
      return 0;
    }
  }
  assert(false);
  return 0;
}
```

## 后记

由于纯暴力的做法实在太慢，复杂度达 $O(n^{4}logn)$，因此只对拍了[1400+组](https://r2-imgs.iliyian.com/img/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202024-10-24%20152312.png) $n=100$ 的数据，并同时对比了 $b$ 和 $c$ 数组的计算结果，~~应该问题不大~~。

总之还是很经典的区间 mex 一类问题，推荐几道类似的：
[Rmq Problem](https://www.luogu.com.cn/problem/P4137)：$O(logn)$ 在线计算某段区间的 mex 的模板题。
[Complicated Computations](https://codeforces.com/contest/1436/problem/E)：同上，还有些 mex 性质。

~~然后这类问题就做完了~~

不知道明天他们能不能做出来（

2024/10/24 17:26:34

## 其他代码

数据生成器代码
```cpp
#include <bits/stdc++.h>
#include <random>

std::mt19937 rd(std::random_device{}());

constexpr int N = 2e3;

int main(int argc, char **argv) {
  int n = N;
  if (argc > 1) {
    if (std::string(argv[1]) == "small") {
      if (argc > 2) n = atoi(argv[2]);
      for (int t = 1; t <= 2000; t++) {
        std::string filename = std::format("small/{:05}_{}.in", t, "small_bf");
        std::fstream fs(filename, std::ios::out);
        fs << n << '\n';
        for (int i = 1; i <= n; i++) {
          fs << std::max(0, std::min<int>(n, std::round(std::exponential_distribution<>{std::uniform_real_distribution<>{0,1}(rd)}(rd)))) << ' ';
        }
        fs << '\n';
      }
      return 0;
    }

    freopen("main.in", "w", stdout);
    n = atoi(argv[1]);
    std::cout << n << '\n';
    for (int i = 1; i <= n; i++) {
      std::cout << std::uniform_int_distribution<>{0, n}(rd) << ' ';
    }
    std::cout << '\n';
    return 0;
  }
  for (int t = 3; t <= 50; t++) {
    std::stringstream ss;
    std::string suf;

    if (t <= 5) {
      suf = "handmade";
    } else if (t <= 30) {
      suf = "uniform";
    } else if (t <= 40) {
      suf = "normal";
    } else {
      suf = "expo";
    }

    std::string filename = std::format("data/{:05}_{}.in", t, suf);
    std::fstream fs(filename, std::ios::out);

    if (t <= 5) {
      if (t == 3) {
        n = N;
        fs << n << '\n';
        for (int i = 1; i <= n; i++) {
          fs << i - 1 << ' ';
        }
      } else if (t == 4) {
        n = N;
        fs << n << '\n';
        for (int i = 1; i <= n; i++) {
          if (i != n) {
            fs << i - 1 << ' ';
          } else {
            fs << 0 << ' ';
          }
        }
      } else if (t == 5) {
        n = N;
        fs << n << '\n';
        for (int i = 1; i <= n; i++) {
          fs << std::min(n, (int)std::round(std::exponential_distribution<>{1}(rd))) << ' ';
        }
      }
      fs << '\n';
      continue;
    } else if (t == 6) {
      n = 1;
    } else if (t == 7) {
      n = 10;
    } else if (t == 8) {
      n = 100;
    } else if (t == 9) {
      n = 1000;
    } else {
      n = N;
    }
    fs << n << '\n';
    for (int i = 1; i <= n; i++) {
      int data;
      if (t <= 30) {
        data = std::uniform_int_distribution<>{0, n}(rd);
      } else if (t <= 40) {
        data = std::round(std::abs(std::normal_distribution<>{0, std::uniform_real_distribution<>{1, 50}(rd)}(rd)));
        data = std::max(0, std::min(n, data));
      } else {
        double lambda = std::uniform_real_distribution<>{0, 1}(rd);
        data = std::max(0, std::min(n, (int)std::round(std::exponential_distribution<>{lambda}(rd))));
      }
      fs << data << ' ';
    }
    fs << '\n';
  }
  return 0;
}
```

纯暴力代码
```cpp
#include <bits/stdc++.h>

int mex(const std::set<int> &x) {
  int ans = 0;
  while (x.count(ans)) {
    ans++;
  }
  return ans;
}

int main(int argc, char **argv) {
  if (argc > 1) freopen(argv[1], "r", stdin);
  if (argc > 2) freopen(argv[2], "w", stdout);
  int n;
  std::cin >> n;
  std::vector<int> a(n + 1);
  for (int i = 1; i <= n; i++) {
    std::cin >> a[i];
  }
  std::vector<std::vector<int>> b(n + 1, std::vector<int>(n + 1));
  std::vector<std::vector<int>> c(n + 1, std::vector<int>(n + 1));
  for (int i = 1; i <= n; i++) {
    for (int j = i; j <= n; j++) {
      std::set<int> x;
      for (int k = i; k <= j; k++) {
        x.insert(a[k]);
      }
      b[i][j] = mex(x);
    }
  }
#ifndef ONLINE_JUDGE
  for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= n; j++) {
      if (j >= i) {
        std::cout << b[i][j] << ' ';
      } else {
        std::cout << "  ";
      }
    }
    std::cout << '\n';
  }
#endif
  std::set<int> s;
  for (int i = 1; i <= n; i++) {
    for (int j = i; j <= n; j++) {
      std::set<int> x;
      for (int l = i; l <= j; l++) {
        for (int r = l; r <= j; r++) {
          for (int k = l; k <= r; k++) {
            x.insert(b[k][r]);
          }
        }
      }
      c[i][j] = mex(x);
      s.insert(c[i][j]);
    }
  }
#ifndef ONLINE_JUDGE
  for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= n; j++) {
      if (j >= i) {
        std::cout << c[i][j] << ' ';
      } else {
        std::cout << "  ";
      }
    }
    std::cout << '\n';
  }
#endif
  std::cout << mex(s) << '\n';
  return 0;
}
```

对拍器代码
```cpp
#include <bits/stdc++.h>
using namespace std;

void iterateFilesInFolder(const std::string& folderPath) {
    for (const auto& entry : filesystem::directory_iterator(folderPath)) {
        std::cout << entry.path() << std::endl;
    }
}

int main(int argc, char **argv) {
  string folder = "data";
  if (argc > 1) folder = argv[1];
  std::vector<std::string> names;
  for (auto &entry : filesystem::directory_iterator(folder)) {
    if (entry.path().extension() == ".in") {
      names.push_back(entry.path().filename().stem().string());
    }
  }
  for (auto &name : names) {
    std::string main = std::format("main.exe {}/{}.in {}/{}.out", folder, name, folder, name);
    system(main.c_str());
    // std::string bf = std::format("bf.exe {}/{}.in {}/{}.ans", folder, name, folder, name);
    // system(bf.c_str());

    // std::string compareCommand = std::format("fc.exe .\\{}\\{}.out .\\{}\\{}.ans", folder, name, folder, name);
    // if (system(compareCommand.c_str()) != 0) {
    //   std::cerr << "Comparison failed for " << name << std::endl;
    //   return 1;
    // }
  }
  return 0;
}
```

2024.10.31 更新：
右上应该是右下，更准确些
