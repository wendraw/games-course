> PPT: [01.数据拟合.pptx](http://staff.ustc.edu.cn/~lgliu/Courses/GAMES102_2020/PPT/GAMES102-1_DataFitting.pdf)
> 视频: [01.数据拟合.mp4](https://www.bilibili.com/video/BV1NA411E7Yr/?share_source=copy_web&vd_source=43929e1eb72edc92ccec94f51d8e77e7)

## 概述

数据拟合是计算机图形学和数值分析中的基础问题，目标是根据给定的数据点找到一个函数来描述或近似这些数据。本项目实现了两种经典的拟合方法：

1. **插值型拟合**：要求拟合函数必须通过所有给定的数据点
2. **逼近型拟合**：允许拟合函数不完全通过数据点，而是寻找最佳的整体拟合效果

---

## 1. 插值型拟合 - 拉格朗日插值法

### 基本原理

拉格朗日插值法是一种经典的多项式插值方法，其基本思想是：给定 n+1 个不同的数据点 $(x_0, y_0), (x_1, y_1), ..., (x_n, y_n)$，构造一个 n 次多项式 $P(x)$，使得 $P(x_i) = y_i$ 对所有 $i = 0, 1, ..., n$ 成立。

### 数学公式

拉格朗日插值多项式的表达式为：

$$P(x) = \sum_{i=0}^{n} y_i \cdot L_i(x)$$

其中 $L_i(x)$ 是拉格朗日基函数：

$$L_i(x) = \prod_{j=0, j \neq i}^{n} \frac{x - x_j}{x_i - x_j}$$

### 算法步骤

1. **输入**：n+1 个数据点 $(x_i, y_i)$
2. **对于每个插值点 x**：
   - 计算每个拉格朗日基函数 $L_i(x)$
   - 计算加权和 $P(x) = \sum_{i=0}^{n} y_i \cdot L_i(x)$
3. **输出**：插值多项式在点 x 处的值

### 代码实现解析

```javascript
interpolation: (points, x) => {
  if (points.length === 0) return 0
  if (points.length === 1) return points[0].y

  let result = 0
  for (let i = 0; i < points.length; i++) {
    let term = points[i].y // y_i
    for (let j = 0; j < points.length; j++) {
      if (i !== j) {
        const denominator = points[i].x - points[j].x
        if (Math.abs(denominator) < 1e-10) {
          return points[i].y // 避免除零错误
        }
        term *= (x - points[j].x) / denominator // 构造 L_i(x)
      }
    }
    result += term
  }
  return result
}
```

### 特点与应用

**优点**：

- 精确通过所有数据点
- 数学形式简洁优美
- 适用于精确插值需求

**缺点**：

- 随着数据点增多，多项式次数增高，可能出现龙格现象（振荡）
- 对数据中的噪声敏感
- 计算复杂度较高：O(n²)

**适用场景**：

- 数据点较少（通常<10 个）
- 数据精度要求高
- 需要平滑连续的曲线

---

## 2. 逼近型拟合 - 最小二乘法

### 基本原理

最小二乘法的核心思想是寻找一个函数（通常是多项式），使得该函数与所有数据点的误差平方和最小。对于二次多项式拟合，我们要找到系数 $a, b, c$ 使得：

$$f(x) = ax^2 + bx + c$$

最小化目标函数：

$$S = \sum_{i=0}^{n} (y_i - f(x_i))^2 = \sum_{i=0}^{n} (y_i - ax_i^2 - bx_i - c)^2$$

### 数学推导

为了最小化 $S$，我们对 $a, b, c$ 求偏导数并令其为零：

$$\frac{\partial S}{\partial a} = -2\sum_{i=0}^{n} x_i^2(y_i - ax_i^2 - bx_i - c) = 0$$

$$\frac{\partial S}{\partial b} = -2\sum_{i=0}^{n} x_i(y_i - ax_i^2 - bx_i - c) = 0$$

$$\frac{\partial S}{\partial c} = -2\sum_{i=0}^{n} (y_i - ax_i^2 - bx_i - c) = 0$$

整理后得到正规方程组：

$$
\begin{cases}
c \cdot n + b \cdot \sum x_i + a \cdot \sum x_i^2 = \sum y_i \\
c \cdot \sum x_i + b \cdot \sum x_i^2 + a \cdot \sum x_i^3 = \sum x_i y_i \\
c \cdot \sum x_i^2 + b \cdot \sum x_i^3 + a \cdot \sum x_i^4 = \sum x_i^2 y_i
\end{cases}
$$

### 矩阵形式

写成矩阵形式：

$$
\begin{bmatrix}
n & \sum x_i & \sum x_i^2 \\
\sum x_i & \sum x_i^2 & \sum x_i^3 \\
\sum x_i^2 & \sum x_i^3 & \sum x_i^4
\end{bmatrix}
\begin{bmatrix}
c \\ b \\ a
\end{bmatrix}
=
\begin{bmatrix}
\sum y_i \\
\sum x_i y_i \\
\sum x_i^2 y_i
\end{bmatrix}
$$

### 高斯消元法求解

使用高斯消元法求解线性方程组：

1. **前向消元**：将矩阵化为上三角矩阵
2. **回代求解**：从最后一行开始逐步求解

### 代码实现解析

```javascript
approximation: (points, x) => {
  // 处理边界情况
  if (points.length === 0) return 0
  if (points.length === 1) return points[0].y
  if (points.length === 2) {
    // 两点时使用线性拟合
    const [p1, p2] = points
    const slope = (p2.y - p1.y) / (p2.x - p1.x)
    return p1.y + slope * (x - p1.x)
  }

  // 计算各项和
  const n = points.length
  let sumX = 0,
    sumY = 0,
    sumX2 = 0,
    sumX3 = 0,
    sumX4 = 0
  let sumXY = 0,
    sumX2Y = 0

  for (let i = 0; i < n; i++) {
    const xi = points[i].x
    const yi = points[i].y
    sumX += xi
    sumY += yi
    sumX2 += xi * xi
    sumX3 += xi * xi * xi
    sumX4 += xi * xi * xi * xi
    sumXY += xi * yi
    sumX2Y += xi * xi * yi
  }

  // 构造系数矩阵
  const matrix = [
    [n, sumX, sumX2, sumY],
    [sumX, sumX2, sumX3, sumXY],
    [sumX2, sumX3, sumX4, sumX2Y],
  ]

  // 求解并返回结果
  const coeffs = gaussElimination(matrix)
  const a = coeffs[2],
    b = coeffs[1],
    c = coeffs[0]
  return a * x * x + b * x + c
}
```

### 特点与应用

**优点**：

- 对噪声数据有较强的鲁棒性
- 可以控制拟合函数的复杂度
- 整体拟合效果良好
- 计算稳定

**缺点**：

- 不一定通过所有数据点
- 需要预先选择拟合函数的形式
- 可能出现过拟合或欠拟合

**适用场景**：

- 数据包含噪声
- 数据点较多
- 需要平滑的趋势线
- 预测和外推

---

## 3. 两种方法的比较

### 视觉对比

在我们的演示应用中：

- 🔵 **蓝色曲线**：插值型拟合 - 精确通过所有红点
- 🟢 **绿色曲线**：逼近型拟合 - 整体趋势拟合，不一定通过所有点

### 数学特性对比

| 特性       | 插值型拟合       | 逼近型拟合                 |
| ---------- | ---------------- | -------------------------- |
| 精度       | 精确通过所有点   | 整体最优，不一定通过所有点 |
| 稳定性     | 对噪声敏感       | 对噪声鲁棒                 |
| 多项式次数 | n-1 次（n 个点） | 可控制（通常低次）         |
| 计算复杂度 | O(n²)            | O(n)                       |
| 适用数据量 | 小数据集         | 大数据集                   |

### 选择建议

**选择插值型拟合当**：

- 数据点少且精确
- 需要精确通过所有点
- 数据无噪声
- 用于精确插值计算

**选择逼近型拟合当**：

- 数据点多
- 数据包含噪声
- 需要平滑趋势
- 用于预测和分析

---

## 4. 实际应用示例

### 在计算机图形学中的应用

1. **曲线设计**：

   - 插值：精确控制关键点位置
   - 逼近：平滑的曲线外观

2. **动画插值**：

   - 关键帧之间的平滑过渡
   - 路径规划

3. **数据可视化**：
   - 散点图的趋势线
   - 数据平滑处理

### 数值分析中的应用

1. **函数逼近**：

   - 复杂函数的多项式近似
   - 数值积分

2. **数据分析**：
   - 实验数据的拟合
   - 趋势预测

---

## 5. 扩展思考

### 高级拟合方法

1. **样条插值**：分段多项式，避免高次多项式的振荡
2. **加权最小二乘**：对不同数据点赋予不同权重
3. **正则化方法**：防止过拟合
4. **非线性拟合**：拟合复杂的非线性函数

### 数值稳定性

1. **条件数**：矩阵求解的数值稳定性
2. **精度控制**：浮点数计算的误差控制
3. **算法优化**：更高效的求解算法

通过本项目的实现，您可以直观地理解这两种基础但重要的拟合方法，为进一步学习更高级的数值方法打下基础。
