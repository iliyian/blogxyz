---
title: 《魔法少女Lya》命题报告
tags: 算法
abbrlink: bf9a8431
date: 2024-09-29 22:46:23
cover: https://r2-imgs.iliyian.com/img/qun.png
top_img: https://r2-imgs.iliyian.com/img/qun.png
---

## 《魔法少女Lya》命题报告

本来是想仿照ioi2024国家集训队论文集里面那一篇《<世界沉睡童话>命题报告》的格式的，但发现就这破题根本没什么深度，根本模仿不了，就这样写吧。
本题部分灵感来自于 $P2572 [SCOI2010] 《序列操作》$，顺便一提在比赛前一天的下午我还临时加强了一下。

### 题目

给定一段01序列，有区间反转和区间赋值的操作，询问区间内最长的01交替的子串。$（1 \le n, m \le 5 \times 10^5）$。

### O(mlogn) 做法

显然区间反转和区间赋值属于线段树模板操作，本题的关键在于线段树中节点的合并。

我的写法是维护节点代表的区间的左/右开始的最长的以0/1开始的区间长度，和最长的以0/1开始的区间长度，pull（或者pushup）时，以从最左开始的以0开始的区间长度为例，如果其长度等于左子节点的总长度，则直接根据左子节点的长度的奇偶性，接上右子节点的相应的从最左开始的以0/1开始的区间长度，0还是1则是奇变偶不变的原则，否则直接继承。其他三个也一样。至于最长的以0/1开始的区间长度，只需要左右子节点取一个max，然后再给中间取一个max即可，中间则是根据于左子节点的右边的值，来选择对应的右子节点的左边的值，使其可以相接，然后就做完了。

确实有点 heavy implementation，我也写了和调了挺久的。

### 代码
```cpp
#include <bits/stdc++.h>
#define lc p << 1
#define rc p << 1 | 1
#define mid (s + t) / 2

constexpr int N = 5e5;

struct Seg {
  int llen0 = 0, llen1 = 0, rlen0 = 0, rlen1 = 0;
  int maxlen0 = 0, maxlen1 = 0;
  int len = 0;
};

std::vector<std::array<int, 2>> tag(N << 2);
std::vector<Seg> seg(N << 2);
std::vector<int> a(N + 1);

Seg merge(const Seg &x, const Seg &y) {
  if (!y.len) return x;
  if (!x.len) return y;
  auto s = Seg{
    x.llen0 == x.len ? x.llen0 + (x.len % 2 == 0 ? y.llen0 : y.llen1) : x.llen0,
    x.llen1 == x.len ? x.llen1 + (x.len % 2 == 0 ? y.llen1 : y.llen0) : x.llen1,
    y.rlen0 == y.len ? y.rlen0 + (y.len % 2 == 0 ? x.rlen0 : x.rlen1) : y.rlen0,
    y.rlen1 == y.len ? y.rlen1 + (y.len % 2 == 0 ? x.rlen1 : x.rlen0) : y.rlen1,
    std::max(x.maxlen0, y.maxlen0),
    std::max(x.maxlen1, y.maxlen1),
    x.len + y.len,
  };
  if (x.rlen0) {
    if (x.rlen0 % 2) {
      s.maxlen1 = std::max(s.maxlen1, x.rlen0 + y.llen1);
    } else {
      s.maxlen0 = std::max(s.maxlen0, x.rlen0 + y.llen1);
    }
  }
  if (x.rlen1) {
    if (x.rlen1 % 2) {
      s.maxlen0 = std::max(s.maxlen0, x.rlen1 + y.llen0);
    } else {
      s.maxlen1 = std::max(s.maxlen1, x.rlen1 + y.llen0);
    }
  }
  return s;
}

void exec(int s, int t, int p, int typ) {
  if (typ == 0) {
    tag[p] = {1, 0};
    seg[p] = {1, 0, 1, 0, 1, 0, seg[p].len};
  }
  if (typ == 1) {
    tag[p][1] ^= 1;
    seg[p] = {seg[p].llen1, seg[p].llen0, seg[p].rlen1, seg[p].rlen0, seg[p].maxlen1, seg[p].maxlen0, seg[p].len};
  }
}

void push(int s, int t, int p) {
  if (tag[p][0]) {
    exec(s, mid, lc, 0);
    exec(mid + 1, t, rc, 0);
  }
  if (tag[p][1]) {
    exec(s, mid, lc, 1);
    exec(mid + 1, t, rc, 1);
  }
  tag[p] = {0, 0};
}

void pull(int p) {
  seg[p] = merge(seg[lc], seg[rc]);
}

void build(int s, int t, int p) {
  seg[p].len = t - s + 1;
  if (s == t) {
    seg[p] = {!a[s], a[s], !a[s], a[s], !a[s], a[s], 1};
    return;
  }
  build(s, mid, lc);
  build(mid + 1, t, rc);
  pull(p);
}

// 只可能区间赋值为0
void reset(int l, int r, int s, int t, int p) {
  if (r < s || l > t)
    return;
  if (l <= s && t <= r) {
    exec(s, t, p, 0);
    return;
  }
  push(s, t, p);
  reset(l, r, s, mid, lc);
  reset(l, r, mid + 1, t, rc);
  pull(p);
}

void rev(int l, int r, int s, int t, int p) {
  if (r < s || l > t)
    return;
  if (l <= s && t <= r) {
    exec(s, t, p, 1);
    return;
  }
  push(s, t, p);
  rev(l, r, s, mid, lc);
  rev(l, r, mid + 1, t, rc);
  pull(p);
}

Seg query(int l, int r, int s, int t, int p) {
  if (l > t || r < s) return {};
  if (l <= s && t <= r) {
    return seg[p];
  }
  push(s, t, p);
  auto sl = query(l, r, s, mid, lc);
  auto rl = query(l, r, mid + 1, t, rc);
  return merge(sl, rl);
}

signed main() {
  std::cin.tie(nullptr)->sync_with_stdio(false);

  int n, m;
  std::cin >> n >> m;
  for (int i = 1; i <= n; i++) {
    std::cin >> a[i];
    a[i] &= 1;
  }
  build(1, n, 1);
  for (int i = 1; i <= m; i++) {
    int op, l, r, x;
    std::cin >> op;
    if (op == 1 || op == 3) {
      std::cin >> l >> r >> x;
      if (x & 1) {
        rev(l, r, 1, n, 1);
      }
    } else if (op == 2) {
      std::cin >> l >> r >> x;
      if (x & 1 ^ 1) {
        reset(l, r, 1, n, 1);
      }
    } else {
      std::cin >> l >> r;
      int len = r - l + 1;
      auto s = query(l, r, 1, n, 1);
      std::cout << std::max(s.maxlen0, s.maxlen1) << '\n';
    }
  }

  return 0;
}
```

### 旧版题目

给定一段01序列，有区间反转和区间赋值的操作，询问区间内的数是否都是01交替的。$（1 \le n, m \le 5 \times 10^5）$。

### 旧版题目 O(mlogn) 做法

区间操作不变，但是维护的只要是节点的区间长度，左侧开始的0/1串长度即可，最后只要判断一下串长度是否等于区间长度即可。

### 旧版题目代码
```cpp
#include <bits/stdc++.h>
#define lc p << 1
#define rc p << 1 | 1
#define mid (s + t) / 2

constexpr int N = 5e5;

struct Seg {
  int llen0 = 0, llen1 = 0;
  int len = 0;
};

std::vector<std::array<int, 2>> tag(N << 2);
std::vector<Seg> seg(N << 2);
std::vector<int> a(N + 1);

Seg merge(const Seg &x, const Seg &y) {
  return Seg{
    x.llen0 == x.len ? x.llen0 + (x.len % 2 == 0 ? y.llen0 : y.llen1) : x.llen0,
    x.llen1 == x.len ? x.llen1 + (x.len % 2 == 0 ? y.llen1 : y.llen0) : x.llen1,
    x.len + y.len,
  };
}

void exec(int s, int t, int p, int typ) {
  if (typ == 0) {
    tag[p] = {1, 0};
    seg[p] = {1, 0, 1};
  }
  if (typ == 1) {
    tag[p][1] ^= 1;
    seg[p] = {seg[p].llen1, seg[p].llen0, seg[p].len};
  }
}

void push(int s, int t, int p) {
  if (tag[p][0]) {
    exec(s, mid, lc, 0);
    exec(mid + 1, t, rc, 0);
  }
  if (tag[p][1]) {
    exec(s, mid, lc, 1);
    exec(mid + 1, t, rc, 1);
  }
  tag[p] = {0, 0};
}

void pull(int p) {
  seg[p] = merge(seg[lc], seg[rc]);
}

void build(int s, int t, int p) {
  seg[p].len = t - s + 1;
  if (s == t) {
    seg[p] = {!a[s], a[s], 1};
    return;
  }
  build(s, mid, lc);
  build(mid + 1, t, rc);
  pull(p);
}

// 只可能区间赋值为0
void reset(int l, int r, int s, int t, int p) {
  if (r < s || l > t)
    return;
  if (l <= s && t <= r) {
    exec(s, t, p, 0);
    return;
  }
  push(s, t, p);
  reset(l, r, s, mid, lc);
  reset(l, r, mid + 1, t, rc);
  pull(p);
}

void rev(int l, int r, int s, int t, int p) {
  if (r < s || l > t)
    return;
  if (l <= s && t <= r) {
    exec(s, t, p, 1);
    return;
  }
  push(s, t, p);
  rev(l, r, s, mid, lc);
  rev(l, r, mid + 1, t, rc);
  pull(p);
}

Seg query(int l, int r, int s, int t, int p) {
  if (l > t || r < s) return {};
  if (l <= s && t <= r) {
    return seg[p];
  }
  push(s, t, p);
  return merge(query(l, r, s, mid, lc), query(l, r, mid + 1, t, rc));
}

signed main() {
//   if (argc > 1) freopen(argv[1], "r", stdin);
//   if (argc > 2) freopen(argv[2], "w", stdout);
  std::cin.tie(nullptr)->sync_with_stdio(false);

  auto start = std::chrono::high_resolution_clock::now();

  int n, m;
  std::cin >> n >> m;
  for (int i = 1; i <= n; i++) {
    std::cin >> a[i];
    a[i] &= 1;
  }
  build(1, n, 1);
  for (int i = 1; i <= m; i++) {
    int op, l, r, x;
    std::cin >> op;
    if (op == 1 || op == 3) {
      std::cin >> l >> r >> x;
      if (x & 1) {
        rev(l, r, 1, n, 1);
      }
    } else if (op == 2) {
      std::cin >> l >> r >> x;
      if (x & 1 ^ 1) {
        reset(l, r, 1, n, 1);
      }
    } else {
      std::cin >> l >> r;
      int len = r - l + 1;
      auto s = query(l, r, 1, n, 1);
      std::cout << (s.llen0 == len || s.llen1 == len ? "YES" : "NO") << '\n';
    }
  }

  return 0;
}
```

### 后记

本来还想着也许有乱搞做法？例如用bitset，也许可以做到 O(mn/w)，所以专门新上传了前三个测试点把nm拉满，还开到了5e5，但直到现在我也没实现，其他人也没有这样写的。不过大概八成是过不了的吧，即使是2e5级别的？

说真的 zhw 赛时 AC 的时候我老开心了，至少证明第一次出的比赛的第一次自己命的题没出锅，他刚开始wa的时候我在课上也忍不住开始查他的代码和我的暴力代码，不过还好最后 AC 了。顺便一提他写的代码比我快了一倍，不知道是快读的关系还是什么，没仔细看。

虽然但是也许把题目写到题目描述里更好？

以及也许在测试数据点加个前缀使其按序测试也好？

至于这场比赛不知道是不是题目乱序的关系，过题数和客观题目难度竟然刚好倒挂了。

而且 tiome 和 rainydyas 在比赛刚结束22秒和1分35秒后刚好过题着实令人惋惜。

至于把题目名改成“明月照积雪”甚至是比赛当天临时起意弄的（x，这样正好，下次就是“朔风劲且哀”，没准下次得写个公告提醒一下难度是乱序的...

2024/9/29 23:25
