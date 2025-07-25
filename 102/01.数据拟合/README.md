> PPT: [01.数据拟合.pptx](http://staff.ustc.edu.cn/~lgliu/Courses/GAMES102_2020/PPT/GAMES102-1_DataFitting.pdf)
> 视频: [01.数据拟合.mp4](https://www.bilibili.com/video/BV1NA411E7Yr/?share_source=copy_web&vd_source=43929e1eb72edc92ccec94f51d8e77e7)

## 数据拟合方法论

1. 确定函数空间 ——“在哪里找拟合函数？”
2. 定义好坏度量 ——“如何判断拟合函数的优劣？”
3. 求解优化问题 ——“如何找到最优函数？”

拟合（Fitting）问题的核心是从观测数据中找到一个函数（或模型），以描述数据背后的潜在规律。其方法论可归纳为 **“三步框架”**：明确函数空间（“在哪里找”）、定义好坏度量（“找哪个好”）、求解优化问题（“怎么找到”）。以下是详细拆解：

### **第一步：确定函数空间——“在哪里找拟合函数？”**

拟合的本质是从“候选函数集合”中挑选合适的函数，这个集合称为 **函数空间**。选择函数空间时需满足两个核心原则：

- **表达能力**：空间内的函数需能“覆盖”数据的潜在规律（即存在函数能较好逼近真实规律）；
- **可操作性**：空间结构需简单，便于后续数学求解（如线性结构、有限参数化）。

#### 常见函数空间选择：

1. **多项式函数空间**  
   由幂函数（$1, x, x^2, ..., x^n$）线性组合构成，即 $f(x) = a_0 + a_1x + a_2x^2 + ... + a_nx^n$。

   - 优点：结构简单，参数少（$n+1$ 个系数 $a_i$），求解方便；
   - 缺点：高次多项式易出现“震荡”（如 Runge 现象），表达能力有限。

2. **三角函数空间**  
   由正弦/余弦函数（$\sin(kx), \cos(kx)$）组合构成（如傅里叶级数），适用于周期性数据（如声波、振动信号）。

3. **样条函数空间**  
   由分段低次多项式（如三次样条）拼接而成，保证整体光滑性（如连续导数），适用于需要“局部光滑”的场景（如曲线建模、机械零件轮廓）。

4. **非线性函数空间**  
   如指数函数（$f(x) = ae^{bx}+c$）、对数函数、神经网络（多层非线性组合）等，适用于复杂非线性规律，但求解难度较高。

### **第二步：定义好坏度量——“如何判断拟合函数的优劣？”**

需明确“拟合函数与数据的接近程度”的量化标准，即**损失函数（Loss Function）**。损失函数的选择取决于数据特性（如是否含噪声）和问题需求（如是否允许误差）。

#### 常见损失函数：

1. **插值损失（误差为零）**  
   要求拟合函数严格经过所有数据点，即 $f(x_i) = y_i$（$x_i, y_i$ 为观测数据）。

   - 适用场景：数据精确无噪声（如工程图纸的离散点）；
   - 缺点：若数据含噪声，会“放大误差”，导致函数震荡。

2. **最小二乘损失（L2 损失）**  
   定义损失为“拟合值与真实值的平方和”：$L = \sum_{i=1}^n [f(x_i) - y_i]^2$，目标是最小化 $L$。

   - 优点：数学性质好（连续可导，线性情况下可解析求解）；
   - 缺点：对异常值（离群点）敏感（平方会放大异常值的影响）。

3. **绝对值损失（L1 损失）**  
   定义损失为“拟合值与真实值的绝对值和”：$L = \sum_{i=1}^n |f(x_i) - y_i|$。

   - 优点：对异常值更稳健（绝对值不会放大极端误差）；
   - 缺点：在零点不可导，求解难度略高（需用线性规划等方法）。

4. **正则化损失**  
   当函数空间复杂（如高次多项式），可能出现“过拟合”（过度贴合训练数据，泛化能力差），需加入正则项限制函数复杂度。例如：  
   $L = \sum [f(x_i) - y_i]^2 + \lambda \sum a_k^2$（$\lambda$ 为正则化系数，惩罚大系数以简化函数）。

### **第三步：求解优化问题——“如何找到最优函数？”**

根据函数空间和损失函数，将拟合问题转化为可求解的数学模型（方程或优化问题），核心是求解函数的参数（如多项式的系数 $a_k$）。

#### 常见求解方法：

1. **解析求解（适用于线性问题）**  
   若函数空间是线性的（如多项式、线性组合的三角函数），且损失为最小二乘，可转化为线性方程组求解。

   - 例：对线性函数 $f(x) = a + bx$，最小二乘损失的最优解满足：  
     $\begin{cases} 
     na + b\sum x_i = \sum y_i \\ 
     a\sum x_i + b\sum x_i^2 = \sum x_i y_i 
     \end{cases}$  
     直接解方程组即可得 $a, b$。

2. **迭代求解（适用于非线性问题）**  
   若函数是非线性的（如 $f(x) = ae^{bx} + c$），或损失函数非二次（如 L1 损失），需用迭代法逐步逼近最优解。

   - 例：梯度下降法（沿损失函数的负梯度方向更新参数）、牛顿法（利用二阶导数加速收敛）等。

3. **数值稳定性处理**  
   当数据存在噪声或函数空间维度较高时，可能出现“病态问题”（如方程组系数矩阵条件数过大，求解不稳定）。需通过：
   - 简化函数空间（降低多项式次数）；
   - 正则化（加入惩罚项）；
   - 数据预处理（去除异常值、标准化）等方法优化。

### **总结：拟合方法论的核心逻辑**

拟合问题的本质是“在合理的函数空间中，用明确的好坏标准，找到最贴合数据规律的函数”。关键决策包括：

- 函数空间需平衡“表达能力”与“简洁性”（避免过拟合/欠拟合）；
- 损失函数需匹配数据特性（如噪声情况、对异常值的容忍度）；
- 求解方法需兼顾“效率”与“稳定性”（解析法优先，复杂场景用迭代法）。

这一框架广泛应用于几何建模（如从离散点恢复曲面）、工程逆向（如零件轮廓重建）、数据分析（如趋势预测）等领域。
