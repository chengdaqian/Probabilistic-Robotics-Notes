# Chapter 02 递归状态估计

## 1. 简介
概率机器人的核心思路，是由传感器数据估计状态。传感器只能获取部分信息，并且有噪声，状态估计就是要从这些数据中恢复出状态变量。概率状态估计算法在可能的状态空间中计算置信度分布。

## 2. 概率的基本概念
建模时，传感器测量、控制、机器人状态等都是**随机变量**，可以取**连续值**（书中大部分技术是在连续空间提出估计、制定决策），并且假定都有概率密度函数（PDF），如一维正态分布的概率密度函数。
&&图1

### 2.1 正态分布
在本书中起到重要作用，缩写为N(x; μ, σ^2)。x经常是一个多为食量，其正态分布为多元的，其概率密度函数为上式的泛化：
&&图2

### 2.2 独立性（Independence）
独立性与其推广（条件独立）在本书中起到重要作用。如果X与Y独立，则P(x|y) = P(x)，也就是Y的信息对求X没有贡献。

### 2.3 贝叶斯准则
**全概率定理**：
&&图3

**贝叶斯准则**：
&&图4
若x由y推测，则

- p(x)为**先验概率分布（prior probability distribution）**  
	总结了综合数据y之前，已经有的关于x的信息
- y为**数据（data）**
- p(x|y)为**后验概率分布（post probability distribution）**


贝叶斯准则可用**逆条件概率**p(y|x)与先验概率p(x)来计算后验概率。此“逆概率”称为生成模型，即假定事件x发生时，得到数据y的概率。

此外，贝叶斯准则的分母p(y)与x无关，即不随后验概率p(x|y)中的x值变化而变化，因此1/p(y)通常写作归一化变量η，即:

	p(x|y) = η * p(y|x) * p(x)

这种归一化写法非常简洁，无需显式计算f(y)，仅使用归一化参数表示最终结果归一为1。

### 2.4 期望

### 2.5 熵
&&图5
熵的概念源于信息论，是x所携带的期望信息，本书中用于机器人信息收集，来表示机器人执行特定行为后可能接收到的信息。

### 2.6 条件独立
上述所有规则适用于以任意随机变量Z为条件的条件概率，如：

- 贝叶斯准则关于Z=z的条件概率
	&&图6
- **条件独立**：p(x|y,z)=p(x|z) * p(y|z)  
	&&图7
	当y不携带任何与x相关的信息，且z的值已知的情况下成立。
	*即，在Z=z的条件下，X与Y是独立的。*    
	条件独立不能推出独立，反之亦不可。但在特殊情况下，两者可能一致。
	
## 3. 机器人环境交互

环境是有内部状态的动态系统，机器人保持着关于环境状态的internal belief。机器人也可以用执行机构影响环境，但影响有些不可预测。

### 3.1 State
环境由**状态**表征，状态是机器人与环境的所有能对未来产生影响的aspects。有动态状态（变化的）与静态状态（不变的）。本书中状态用x表示，典型状态如下：

- 机器人pose，也叫kinematic state。
- 机器人执行机构的configuration，如机械臂的每一个自由度都是一个一维configuration。属于pose的一部分。
- 机器人与其关节的速度，称动态状态。
- 环境中物体的位置和特征。特征如外观（颜色、纹理）。有些时候假设这些物体为landmark，他们即明显可分辨，又静止不动。

**State Completeness**: complete state是预测未来的最佳手段，加入任何其他信息都不能更好地预测。实践中不可能获得complete state，因此只选择状态变量中的一部分，称incomplete state。

**state变量可连续可离散**，若state spaces包含两者，称hybrid state spaces。本书中时间是离散的。

### 3.2 环境交互

- 机器人改变环境（通过执行器）
- 机器人获取环境信息（通过传感器）

#### 传感器测量
使用传感器获取环境信息的过程叫**感知（perception）**，感知交互的结果称**测量(measurement)**，也称percept/observation. 

#### 控制动作 control action
改变环境的状态，例如机器人运动、操作环境物体等。假设机器人永远都在执行控制动作。

**数据data**: 机器人所有过往的**传感器测量**与**控制动作**，分别为：

- **环境测量数据** Measurement data  
	保存了环境的暂态信息，记为z<sub>t</sub>。z<sub>t1:t2</sub> = z<sub>t1</sub>, z<sub>t1+1</sub>, ..., z<sub>t2</sub>  
- **控制数据** Control data  
	保存了环境中state的变化，如机器人的速度。记为u<sub>t</sub>。里程计也看做控制数据。

Perception增加环境状态的信息，而Motion会丢失信息。两者同时发生，区分两者是为了方便。

### 3.3 概率生成法则
state和measurement的演变具有一定随机性，因此需要用概率表示。

####state
假设机器人先执行u<sub>1</sub>，再获取meassurement z<sub>1</sub>   
*p*(x<sub>t</sub> | x<sub>0:t-1</sub>,z<sub>1:t-1</sub>,u<sub>1:t</sub>)

如果state是complete的，则x<sub>t-1</sub>对于时间1:t-1提供的信息足够，上式变为：  
*p*(x<sub>t</sub> | x<sub>0:t-1</sub>,z<sub>1:t-1</sub>,u<sub>1:t</sub>) = *p*(x<sub>t</sub> | x<sub>t-1</sub>,u<sub>t</sub>)

上式满足**条件独立**的关系，也就是x<sub>t</sub>与x<sub>1:t-2</sub>, z<sub>1:t-1</sub>, u<sub>1:t-1</sub>是独立的

式*p*(x<sub>t</sub> | x<sub>t-1</sub>,u<sub>t</sub>)称**状态转移概率**（state transition probability）。若不依赖时间，则可写成*p*(x' | u, x)，x'为次态，x为现态。

####measurement

同理，有  
*p*(z<sub>t</sub> | x<sub>0:t</sub>, z<sub>1:t-1</sub>, u<sub>1:t</sub>) = *p*(z<sub>t</sub> | x<sub>t</sub>)

式*p*(z<sub>t</sub> | x<sub>t</sub>)称**测量概率**（measurement probability）。若不依赖时间，则可写成*p*(z | x)。表明，测量z是由环境状态x产生的，可以将测量认作状态的有噪声预测。

####总结
状态转移概率与测量概率以其描述机器人及其环境组成的动态随机系统。时刻t的状态依赖t-1时刻的状态与控制u<sub>t</sub>，测量z<sub>t</sub>依赖时刻t的状态。这种**时间生成模型**也称为隐马尔科夫模型（Hidden Markov Model, HMM）或动态贝叶斯网络（Dynamic Bayes Network, DBN）

### 3.4置信分布 Belief Distributions
**Belief反应了机器人关于环境状态的内部知识**。概率机器人学使用**条件概率分布**来表示belief，对于每一个真实state，不同的假设有不同的概率。belief distribution是**后验概率**，记为：  
*bel(x<sub>t</sub>)* = *p(x<sub>t</sub> | z<sub>1:t</sub> , u<sub>1:t</sub> )*

有时也使用执行后、测量前（没有z<sub>t</sub>）的后验概率，记为：  
<SPAN style='TEXT-DECORATION:overline'>*bel*</SPAN> = *p(x<sub>t</sub> | z<sub>1:t-1</sub> , u<sub>1:t</sub> )*  
这个概率分布也被称作**prediction**。从<SPAN style='TEXT-DECORATION:overline'>*bel*</SPAN>计算*bel*也叫**correction**或**measurement update**。

##4. 贝叶斯滤波器 Bayes Filters

###4.1 贝叶斯滤波器算法
用measurement和control计算belief distribution。本算法是recursive的：用*bel(x<sub>t-1</sub>), u<sub>t</sub>, z<sub>t</sub>*来计算*bel(x<sub>t</sub>)*。

递归Update算法：

- **用*bel*(x<sub>t-1</sub>)、*u<sub>t</sub>*计算<SPAN style='TEXT-DECORATION:overline'>*bel*</SPAN>**：<SPAN style='TEXT-DECORATION:overline'>*bel*</SPAN>=*p*(xt | ut,xt-1)*bel*(xt-1)*dx*  
	用之前的belief distribution以及融合ut的状态转移概率（见3.3）。
- **用<SPAN style='TEXT-DECORATION:overline'>*bel*</SPAN>**
