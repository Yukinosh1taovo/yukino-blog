+++
date = '2026-01-14T14:19:15+08:00'
draft = true
title = 'python数据分析学习'

+++

# python数据分析学习



## 基础操作

### 1. Anaconda Prompt操作(命令行)

``` bash
conda (list) env # 查看当前命令来源于哪些包 (如list)
conda env list # 查看已部署沙箱
conda activate 沙箱名 # 切换到沙箱名指定的沙箱
conda deactivate 沙箱名 # 退出当前沙箱
conda create [-n](env_name) python=VERSION_CODE # -n选项指定创建沙箱名  python指定版本
conda remove [-n](env_name) [--all] # 删除指定沙箱
conda search 包名 # 搜索指定包信息
conda list 包名 # 查询本机已安装该包信息

```

---

### 2. jupyter notebook快捷键 (Pycharm环境)

**命令模式下**

```
A   在当前cell上方新建cell
B   在当前cell下方新建cell
Y   切换到代码模式
M   切换到markdown模式
DD  删除当前cell
```

**编辑模式下**

```
ctrl + shift + enter 切换到下一行
```

**通用模式下**

```
ctrl + enter 执行当前cell并停留在当前cell
shift + enter 执行当前cell并转到下一cell
```

---

## Numpy

NumPy数组是一个多维的数组对象（矩阵），称为ndarray，具有矢量算术运算能力和复杂的广播能力，并具有执行速度快和节省空间的特点。注意：ndarray的下标从0开始，且数组里的所有元素必须是相同类型。

### 1. Numpy导入

```python
import numpy as np  # 导入numpy库
arr = np.array( [ARRAY_SAMPLE] ) # 使用numpy的array方法创建一个 ndarray 类型的数组对象
```

### 2. ndarray类型对象的基本属性

```python
arr.ndim      # number of dimensions 数组的维度
arr.shape     # 数组的行数和列数
arr.size      # 数组的总元素个数
arr.dtype     # 数组的元素类型
arr.itemsize  # 数组的单个元素占用内存字节数
```

### 3. numpy库函数

静态方法

```python
# 创建ndarray对象相关
np.array( [LIST] ) # 将list类型对象转为ndarray类型对象
np.arange(起始, 结束, 步长, dtype = 类型) # 按左闭右开范围按步长创建一位数组 .reshape(row, col) 可指定生成指定多维数组
np.random.rand(row, col) # 生成指定行列数的0.0 ~ 1.0之间(左闭右开)的随机小数矩阵
np.random.randint(起始, 结束, size = (row, col)) # 生成指定行列的指定范围的随机整数矩阵
np.random.uniform(起始, 结束 size = (row, col)) # 生成指定范围 左闭右开区间的随机小数矩阵
np.random.randn() # 创建标准正态分布序列
np.logspace(起始, 结束, 元素个数, base = 底数) # 生成以底数的起始幂到结束幂(左闭右闭)的等比数列 如np.logspace(0, 4, 5, base=2) 默认为浮点数
										 									   # 结果为[1, 2, 4, 8, 16]
np.linespace(起始, 结束, 元素个数) # 生成包含起始值和结束值(左闭右闭)的等差数列  可以指定endpoint决定是否包含结束值 默认为浮点数 dtype可指定元素类型

# 处理arr元素值相关
np.ceil(arr)  # 将arr内元素值向上取整
np.floor(arr) # 将arr内元素值向下取整
np.abs(arr)   # 将arr内元素值取绝对值
np.multiply(arr1, arr2) # 将arr1和arr2对应位置元素相乘(行列数一致) 效果同 arr1 * arr2
np.divide(arr1, arr2)   # 将arr1和arr2对应位置元素相除(行列数一致)
np.where(arr > 0, 1, -1) # 将arr中元素依次代入三目表达式 
np.cumsum(arr) # 计算arr的前缀和数组
np.sum( arr [, axis = op] ) # 对arr元素求和 返回一个数  可选axis参数 op=0 时对列求和 返回长度为列数的数组  op=1 时对行求和
np.sort(arr) # 对arr排序 返回副本
np.unique(arr) # 返回对arr去重后的副本(一维)

# 矩阵运算
np.dot(arr1, arr2)  arr1.dot(arr2)   arr1 @ arr2 # 矩阵运算
```

成员方法

```python
arr.astype(np.float32) # 将arr对象中的元素转为float32类型
arr.T # 将arr转置
arr.sort() # arr直接排序
```



## Pandas

导包

```python
import numpy as np
import pandas as pd
```

### 1. 创建Series对象

Series也是Pandas中的最基本的数据结构对象，下文中简称s对象；是DataFrame的列对象，series本身也具有索引。

```python
s1 = pd.Series([1,2,3,4,6]) # 采用默认自增索引
s2 = pd.Series([1,2,3,4,6], index = ['a', 'b', 'c', 'd', 'e'])
s3 = pd.Series((11,22,33,44)) # 使用元组创建
s4 = pd.Series( {'a': 1, 'b' : 2, 'c': 3} ) # 使用字典创建
s5 = pd.Series(np.arange(5)) # 使用numpy数组对象创建
```





## Matplotlib

 

## Seaborn



## Sklearn

