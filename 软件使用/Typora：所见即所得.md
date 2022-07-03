## Typora：所见即所得

> 记录Markdown的使用语法，并结合Typora进行编辑；很早之前的一篇笔记，不断添加内容...目前作为个人的一个参考文件使用。

### 常用快捷键

- 加粗：`ctrl + B`
- 标题：`ctrl + 1~6`，对应于1~6级标题
- 插入公式：`ctrl + Shift + m`
- 插入代码：`ctrl + Shift + K`
- 插入图片：`ctrl + Shift  + I`
- 查看大纲：`ctrl + Shift + L`

------

### 块元素

#### 标题级别

- `#+space+1`一级标题，快捷键为`ctrl + 1`
- ... ...
- `#+space+6`六级标题，快捷键为`ctrl + 6`

#### 清单

* 无序列表：

  `*+空格+内容`

* 有序列表

  `1.+空格+内容`

> 撤销回到上一级： Shift+tab键/回车键

#### 引用文字

`>+空格+引用内容`

> 这是效果图

#### 表格

- 方式一：表格内容需要顶头写，格式如下： |表头1|表头2|...  +回车
- **方式二：右击->插入->表格**

#### 脚注

- 创建脚注`[^1]`

> 从未使用过，具体用法待探究

#### 分割线

- 输入`***`/`---`再按回车即可绘制一条水平分割线

#### 目录

- 输入`[toc]`然后回车，即可创建一个“目录”

#### 加粗倾斜

- 在内容前后加`两个*`为加粗
- 在内容前后加`一个*为倾斜

#### 代码标记

- `这是效果`

  ```
  `print（）`  `——在英文输入法下，用~这个按键
  ```

#### 链接

* 不带标题的链接：`[显示名字](链接)`
* 带标题的链接：`[显示名字](链接 "标题")`

[连接到博客主页](https://xianyuchao.cn/)

***

### 公式添加

**快捷键 `ctrl + Shift + m`** 

#### 上下标

* 上标：`x^{i}`——$x^{i}$
* 下标：`y_{i}`——$y_{i}$

#### 分式

* `\frac{1}{2}`——$\frac{1}{2}$

#### 根式

* `\sqrt{2}`——$\sqrt{2}$
* `\quad\sqrt[p]{2}`——$\quad\sqrt[p]{2}$

#### 积分

* `不带上下标：\int{x}dx`——$\int{x}dx$
* `带上下标：\int_{1}^{2}{x}dx`——$\int_{1}^{2}{x}dx$

#### 极限

* `\lim{a+b}`——$\lim{a+b}$
* `\lim_{n\rightarrow+\infty}`——$\lim_{n\rightarrow+\infty}$

#### 求和

* 不带上下标：`\sum{a}`——$\sum{a}$
* 带上下标：`\sum_{n=1}^{100}{a_n}`——$\sum_{n=1}^{100}{a_n}$

#### 累乘

* 不带上下标：`\prod{x}`——$\prod{x}$
* 带上下标：`\prod_{n=1}^{99}{x_n}`——$\prod_{n=1}^{99}{x_n}$

#### 三角函数

* sin：`\sin`——$\sin$

#### 对数函数

* ln2：`\ln2`——$\ln2$
* log28(注：2是底数)：`\log_28`——$\log_28$
* lg10：`\lg10`——$\lg10$

#### 上标

* 向量：`\vec{a}`——$\vec{a}$
* 平均值：`\overline{a}`——$\overline{a}$
* 预测值(尖尖头)：`\hat{a}`——$\hat{a}$
* 一阶导数(小点)：`\dot{a}`——$\dot{a}$
* 二阶导数(小点点)：`\ddot{a}`——$\ddot{a}$

#### 矩阵

> 渲染后有点问题（时好时不好），但本地看是没问题的...

* 带方括号的矩阵：`\left[\begin{matrix} a & b & c & d & e \\ f & g & h & i & j \\ k & l & m & n & o \\ p & q & r & s & t \end{matrix} \right]`

$$
\left [ \begin{matrix} a & b & c & d & e \\ f & g & h & i & j \\ k & l & m & n & o \\ p & q & r & s & t \end{matrix} \right]
$$

* 画带大括号的矩阵：`\left\{ \begin{matrix} a & b & c & d & e\\ f & g & h & i & j \\ k & l & m & n & o \\ p & q & r & s & t \end{matrix} \right\}`

$$
\left \{ \begin{matrix} a & b & c & d & e \\ f & g & h & i & j \\ k & l & m & n & o \\ p & q & r & s & t \end{matrix} \right\}
$$

* 矩阵中间有省略号：`A= \left\{ \begin{matrix} a & b & \cdots & e\\ f & g & \cdots & j \\ \vdots & \vdots & \ddots & \vdots \\ p & q & \cdots & t \end{matrix} \right\}`
  $$
  A= \left\{ \begin{matrix} a & b & \cdots & e \\ f & g & \cdots & j \\ \vdots & \vdots & \ddots & \vdots \\ p & q & \cdots & t \end{matrix} \right\}
  $$

  > `\cdots`为水平方向的省略号
  >
  > `\vdots`为竖直方向的省略号
  >
  > `\ddots`为斜线方向的省略号

#### 希腊字母

| 输入     | MarkDown | 输入        | MarkDown |
| -------- | -------- | ----------- | -------- |
| \Delta   | Δ        | \mu         | μ        |
| \Theta   | Θ        | \tau        | τ        |
| \Lambda  | Λ        | \phi        | ϕ        |
| \Sigma   | Σ        | \varphi     | φ        |
| \Omega   | Ω        | \psi        | ψ        |
| \Psi     | Ψ        | \omega      | ω        |
| \lambda  | λ        | \pi         | π        |
| \theta   | θ        | \xi         | ξ        |
| \eta     | η        | \gamma      | γ        |
| \alpha   | α        | \delta      | δ        |
| \beta    | β        | \zeta       | ζ        |
| \epsilon | ϵ        | \varepsilon | ε        |

#### 关系运算符

* ±：`\pm`
* x：`\times`
* ⋅ ：`\cdot`
* ÷：`\div`
* ≠：`\neq`
* ≡：`\equiv`
* ≤：`\leq`
* ≥：`\geq`

#### 花括号

* `c(u)=\begin{cases} \sqrt\frac{1}{N}，u=0\\ \sqrt\frac{2}{N}， u\neq0\end{cases}`，效果如下：

$$
c(u)=\begin{cases} \sqrt\frac{1}{N}，u=0\\ \sqrt\frac{2}{N}， u\neq0\end{cases}
$$

#### 其他特殊字符

| 符号 | Markdown          |
| ---- | ----------------- |
| ∀    | \forall           |
| ∞    | \infty            |
| ∅    | \emptyset         |
| ∃    | \exists           |
| ∇    | \nabla            |
| ⊥    | \bot              |
| ∠    | \angle            |
| ∵    | \because          |
| ∴    | \therefore        |
| 空格 | \space或者’~‘符号 |

***

### 参考

[参考链接](https://www.simon96.online/2018/10/18/Typora%E5%85%A5%E9%97%A8%EF%BC%88%E4%B8%AD%E6%96%87%E7%89%88%EF%BC%89/)

[公式参考链接1](https://blog.csdn.net/mingzhuo_126/article/details/82722455)

[公式参考链接2](https://www.jianshu.com/p/756bc7e0ef6d)

