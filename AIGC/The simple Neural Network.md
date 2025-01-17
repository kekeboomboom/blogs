# The simple Neural Network

Status: In progress

神经网络很复杂，用最简单基础的模块看看他在做什么。

用最简单的单层感知器学习最基本的神经网络知识。

例子：我们这是一个二元的感知器，用来判断邮箱中的邮件是否是垃圾邮件。x1 表示邮件中包含“免费”的次数，x2 表示邮件中包含“会议”的次数。y 表示邮件是否是垃圾邮件。

## 一些类比

我从 ChatGPT 里面的一个 GPTS 去问了一下基础神经网络的信息。他使用类比法来讲解神经网络中的每个概念。他给的例子是一个人是否决定出门。

是否决定出门是输出。

输入是外面天气是否雨天，自己是否有雨伞，外面是否冷。

权重，是每个输入因素对于此人的心理占比。比如有人只要下雨就不想出门，那么下雨这个权重就很高。

偏置，比如现在天气到了冬天，天气很冷，无论外面是否下雨或者其他怎样，你都不想出门。那么此时偏置就会更加偏向不出门。

激活函数，对于简单的二元问题，比如最终决定是否出门。通过输入、权重、偏置，经过一个函数运算得到一个预测值。最终这个值通过激活函数归到 0 或者 1，来判断是否决定出门。

## **How a Neural Network Learns**

上面只是一些概念，下面是完整流程。

1. **Forward Pass**: 就是正向训练，通过权重、偏置、输入值，通过一个函数计算出预测值。
2. **Error Calculation (Loss)**: 计算预测值与实际值的误差
3. **Backpropagation**: 我们计算出了预测值，但是预测值与真实值有误差，我们想要模型预测的更准，那么我们希望能够反向通过一定的手段修改权重和偏置，来让下一次预测更准。那么常用的方法是梯度下降法。关于梯度下降法很好的一篇 blog: https://towardsdatascience.com/gradient-descent-algorithm-a-deep-dive-cf04e8115f21. 我们训练的目的就是降低误差，因此我们需要得到一个关于误差的函数，他们在机器学习中，这个函数叫做损失函数，常用的损失函数有：**均方误差 (MSE)**
   $$
    L = \frac{1}{n} \sum_{i=1}^{n} (y_i - y_{pred_i})^2
    $$
    
    我们希望得到的是损失函数关于权重和偏置的函数，通过修改权重和偏置，来降低损失函数值。可以将权重和偏置视为变量。那么损失函数关于权重的偏导数：
    
    $$
    \frac{\partial L}{\partial w_i} = \frac{\partial}{\partial w_i}\left(\frac{1}{n}\sum_{i=1}^{n}(y_i - y_{pred_i})^2\right)
    $$
    
    其中:
    
    $$
    y_{pred_i} = f(w_1x_{1i} + w_2x_{2i} + b)
    $$
    
    使用链式法则计算偏导数（**如果你难以看出如何计算出来的，可以想想 f(x) = (a - bx)² ，a 和 b 都算常数，算此函数的导数：f’(x) = -2 * (a-bx) * b** ）:
    
    $$
    \frac{\partial L}{\partial w_i} = -\frac{2}{n}\sum_{i=1}^{n}(y_i - y_{pred_i}) \cdot x_i
    $$
    
    损失函数相对于偏置的梯度：
    
    $$
    \frac{\partial L}{\partial b} = \frac{\partial}{\partial b}\left(\frac{1}{n}\sum_{i=1}^{n}(y_i - y_{pred_i})^2\right)
    $$
    
    使用链式法则：
    
    $$
    \frac{\partial L}{\partial b} = -\frac{2}{n}\sum_{i=1}^{n}(y_i - y_{pred_i})
    $$
    
4. **Updating Weights**: 如果将权重和偏置视为自变量，我们希望能够通过修改权重和偏置，来让损失函数的值更小。我们可以通过梯度下降法来进行搜索。第 3 步我们已经得到关于权重和偏置的偏导数，那么他们的更新规则如下:
   
    $$
    w_i = w_i - \eta \cdot \frac{\partial L}{\partial w_i}
    $$
    
    $$
    b = b - \eta \cdot \frac{\partial L}{\partial b}
    $$
    
    其中 η 是学习率，控制更新的步长。
    

---

上面是关于神经网络的一些基本概念，接下来通过一些实际数据来具体感受一下。比如还是判断垃圾邮件，x1 表示“免费体验”字段出现的频率，x2 表示“会议”字段出现的频率，y 表示是否垃圾邮件。

## 输入集

| **x1** | **x2** | **y** |
| --- | --- | --- |
| 2 | 1 | 1 |
| 0 | 3 | 0 |
| 1 | 2 | 1 |
| 3 | 0 | 1 |
| 0 | 1 | 0 |

## 初始参数

权重：每个特征（`x1` 和 `x2`）都有一个对应的权重（`w1` 和 `w2`）。这些权重在训练过程中调整，以反映特征对分类结果的重要性。

偏置：偏置（`b`）允许模型在没有输入特征的情况下也能做出预测。在垃圾邮件分类中，偏置可以帮助模型在特征不明显时做出默认判断。

学习率：η 的选择对梯度下降算法的收敛速度和稳定性有重要影响。如果学习率太大，可能会导致模型在最小值附近振荡；如果学习率太小，模型收敛速度会很慢。

参数初始化

- 权重: w1=0.1, w2=0.1
- 偏置: b=0
- 学习率: η=0.1

## 激活函数

使用阶跃激活函数：

$$
f(x) = \begin{cases} 1 & \text{if } z >= 0 \\ 0 & \text{if z < 0} \end{cases}
$$

## 计算步骤

计算预测值公式：**z = w1x1 + w2x2 + b**

计算误差或者说叫做损失函数：**e = y - y_pred （**y_pred就是上面我们计算得出的预测值**）**

### **计算所有样本的线性输出和预测值**：

| **x1** | **x2** | **y** | **z = w1*x1 + w2*x2 + b** | **y_pred** |
| --- | --- | --- | --- | --- |
| 2 | 1 | 1 | 0.1*2 + 0.1*1 + 0 = 0.3 | 1 |
| 0 | 3 | 0 | 0.1*0 + 0.1*3 + 0 = 0.3 | 1 |
| 1 | 2 | 1 | 0.1*1 + 0.1*2 + 0 = 0.3 | 1 |
| 3 | 0 | 1 | 0.1*3 + 0.1*0 + 0 = 0.3 | 1 |
| 0 | 1 | 0 | 0.1*0 + 0.1*1 + 0 = 0.1 | 1 |

### **计算所有样本的误差**：

| **x1** | **x2** | **y** | **y_pred** | **e = y - y_pred** |
| --- | --- | --- | --- | --- |
| 2 | 1 | 1 | 1 | 0 |
| 0 | 3 | 0 | 1 | -1 |
| 1 | 2 | 1 | 1 | 0 |
| 3 | 0 | 1 | 1 | 0 |
| 0 | 1 | 0 | 1 | -1 |

### **计算梯度**：

$$
\frac{\partial L}{\partial w_1} = -\frac{2}{n}\sum_{i=1}^{n}(y_i - y_{pred_i}) \cdot x_{1i} = -\frac{2}{5}[0 \cdot 2 + (-1) \cdot 0 + 0 \cdot 1 + 0 \cdot 3 + -1 \cdot 0] = 0
$$

$$
\frac{\partial L}{\partial w_2} = -\frac{2}{n}\sum_{i=1}^{n}(y_i - y_{pred_i}) \cdot x_{2i} = -\frac{2}{5}[0 \cdot 1 + (-1) \cdot 3 + 0 \cdot 2 + 0 \cdot 0 + (-1) \cdot 1] = \frac{8}{5}
$$

$$
\frac{\partial L}{\partial b} = -\frac{2}{n}\sum_{i=1}^{n}(y_i - y_{pred_i}) = -\frac{2}{5}[0 + (-1) + 0 + 0 + (-1)] = \frac{4}{5}
$$

### **更新权重和偏置**：

$$
w_1 = w_1 - \eta \cdot \frac{\partial L}{\partial w_1} = 0.1 - 0.1 \cdot 0 = 0.1
$$

$$
w_2 = w_2 - \eta \cdot \frac{\partial L}{\partial w_2} = 0.1 - 0.1 \cdot \frac{8}{5} = -0.06
$$

$$
b = b - \eta \cdot \frac{\partial L}{\partial b} = 0 - 0.1 \cdot \frac{4}{5} = -0.08
$$

### **结果**

经过一次标准梯度下降迭代后，更新的参数为：

- w1=0.1
- w2=−0.06
- b=−0.08

这些参数将用于下一次迭代，直到模型收敛。

## Summary

以上就是对于一个二维的变量的一个神经网络去学习的过程。当然我们一个把变量控制在一维，那样更简单，更易理解。但是二维是一个更适中的维度，因为现实中会有更多的变量，这些不同维度，不同的数据，经过优化算法的处理，让优化出来的参数能更加好的预测，这能帮助我们更好的理解一些事情。

关于其中的扩展点，现在只是一个单个神经节点的学习过程，那么对于一个真正的网络是怎样的？

隐藏层是什么？

还有那些优化方法，梯度下降法只是其中的一个？不同的方法有什么区别？这对于我们学习使用 AI 有什么帮助？

如果你想要做一个 AI 的 demo，或者已经做好了一个 AI demo，想要更进一步的优化，我们应该怎么优化？对于个人开发者来说，更便宜的、资源有限的人来说，该如何优化？