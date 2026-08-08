# 计算几何基础

!!! info "考纲定位"
    计算几何通常属于 NOI 级拓展。本页先建立向量、叉积和面积等基础，不把它们作为 NOIP/CSP-S 必修模板。

计算几何最容易出错的地方不是公式数量，而是对象含义、方向和数值类型。一个稳定的做法是：统一用点表示位置，用两个点之差表示向量，再把方向判断集中到点积和叉积。

## 点与向量

点 $P=(x,y)$ 表示位置；向量 $\vec v=(x,y)$ 表示方向与长度。代码上二者可以共用结构体，但运算含义不同：

- `pointB - pointA` 得到从 $A$ 指向 $B$ 的向量；
- 向量可以相加、缩放；
- 两个位置相加通常没有直接几何意义。

```cpp
struct Point {
    long long x, y;
    Point operator+(const Point& other) const {
        return {x + other.x, y + other.y};
    }
    Point operator-(const Point& other) const {
        return {x - other.x, y - other.y};
    }
};
```

若坐标是整数且只做方向、相交和面积判断，应尽量保持整数运算，避免不必要的浮点误差。

## 点积：投影与夹角

两个向量的点积为：

$$
\vec a\cdot\vec b=a_xb_x+a_yb_y=|a||b|\cos\theta.
$$

因此：

- 点积 $>0$：夹角为锐角；
- 点积 $=0$：两向量垂直；
- 点积 $<0$：夹角为钝角。

判断点 $P$ 在线段 $AB$ 所在直线上的投影是否落在线段内，可以检查：

$$
(P-A)\cdot(P-B)\le0.
$$

## 叉积：方向与有向面积

二维向量的叉积是标量：

$$
\vec a\times\vec b=a_xb_y-a_yb_x.
$$

它等于两向量张成平行四边形的有向面积：

- $>0$：从 $\vec a$ 转到 $\vec b$ 是逆时针；
- $<0$：顺时针；
- $=0$：共线。

```cpp
long long cross(Point a, Point b) {
    return a.x * b.y - a.y * b.x;
}

long long orientation(Point a, Point b, Point c) {
    return cross(b - a, c - a);
}
```

`orientation(A,B,C)` 判断从有向线段 $AB$ 转向 $AC$ 的方向。

!!! warning "乘积也会溢出"
    坐标绝对值若达到 $10^9$，两个坐标相乘接近 $10^{18}$，差值可能超过 `long long`。应根据数据范围改用 `__int128` 计算叉积。

## 点在线段上

点 $P$ 在线段 $AB$ 上，需要同时满足：

1. 三点共线；
2. $P$ 的坐标处于 $A,B$ 的包围盒内。

```cpp
bool onSegment(Point a, Point b, Point p) {
    return orientation(a, b, p) == 0 &&
           min(a.x, b.x) <= p.x && p.x <= max(a.x, b.x) &&
           min(a.y, b.y) <= p.y && p.y <= max(a.y, b.y);
}
```

只检查共线会把线段延长线上的点也误判为在线段上。

## 两线段是否相交

线段 $AB$ 与 $CD$ 严格跨立时：$C,D$ 位于直线 $AB$ 两侧，并且 $A,B$ 位于直线 $CD$ 两侧。

还要单独处理端点接触与共线重叠：若某个方向值为 0，再用 `onSegment` 判断该点是否落在另一条线段上。

```cpp
int sign(long long value) { return (value > 0) - (value < 0); }

bool segmentsIntersect(Point a, Point b, Point c, Point d) {
    long long abC = orientation(a, b, c);
    long long abD = orientation(a, b, d);
    long long cdA = orientation(c, d, a);
    long long cdB = orientation(c, d, b);

    if (sign(abC) * sign(abD) < 0 && sign(cdA) * sign(cdB) < 0)
        return true;
    return (abC == 0 && onSegment(a, b, c)) ||
           (abD == 0 && onSegment(a, b, d)) ||
           (cdA == 0 && onSegment(c, d, a)) ||
           (cdB == 0 && onSegment(c, d, b));
}
```

## 多边形面积：鞋带公式

按顺时针或逆时针顺序给出多边形顶点 $P_0,P_1,\ldots,P_{n-1}$。把每条边与原点组成的有向三角形面积相加：

$$
2S=\left|\sum_{i=0}^{n-1}P_i\times P_{(i+1)\bmod n}\right|.
$$

取绝对值前的结果还能反映顶点方向；逆时针通常为正，顺时针为负。

### 手算例子

矩形顶点依次为 $(0,0),(3,0),(3,2),(0,2)$：

$$
2S=0+6+6+0=12,
$$

因此面积为 6。

## 真题：P1183 多边形的面积

[洛谷 P1183 多边形的面积](https://www.luogu.com.cn/problem/P1183) 按逆时针给出轴对齐简单多边形顶点。轴对齐不是使用鞋带公式的必要条件，只是保证本题面积为整数。

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Point { long long x, y; };

long long cross(Point a, Point b) {
    return a.x * b.y - a.y * b.x;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;
    vector<Point> polygon(n);
    for (Point& point : polygon) cin >> point.x >> point.y;

    long long doubledArea = 0;
    for (int i = 0; i < n; ++i)
        doubledArea += cross(polygon[i], polygon[(i + 1) % n]);
    cout << llabs(doubledArea) / 2 << '\n';
    return 0;
}
```

只扫描一遍顶点，时间 $O(n)$、额外空间 $O(n)$；若边读边保留首点和前一点，可降为 $O(1)$ 额外空间。

## 浮点计算约定

涉及距离、交点坐标或圆时通常需要 `double`。不要直接用 `a == b`，而是根据数据尺度选择 `EPS`：

```cpp
const double EPS = 1e-9;
int sign(double value) {
    if (value > EPS) return 1;
    if (value < -EPS) return -1;
    return 0;
}
```

`EPS` 不是越小越好，也不能修复已经溢出的整数乘法。先选择正确数值类型，再讨论误差比较。

## 易错点

- 把 `cross(A,B)` 与 `cross(B-A,C-A)` 混为一谈。
- 只判断方向相反，漏掉端点接触与共线重叠。
- 认为共线就一定在线段内，忘记包围盒。
- 鞋带公式漏掉最后一个顶点到第一个顶点的边。
- 整数乘法溢出后才转换为 `long long` 或 `double`；转换必须发生在乘法之前。
