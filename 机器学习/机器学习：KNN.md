## 机器学习：KNN

> KNN / K-nearst neighbors / K-近邻；即为一种**物以类聚，人以群分**的思想。

### 简介

> KNN：K-nearst neighbors

* k-近邻算法采用测量不同特征值之间的距离来进行分类，简而言之为：**物以类聚，人以群分**
* KNN既可以应用于分类中，也可用于回归中；在分类的预测是，一般采用**多数表决法**；在做回归预测时，一般采用**平均值法**

### KNN三要素

在KNN的算法中，主要考虑以下三个要素：

* **K值的选择**：表示样本可由距离其最近的K个邻居来代替；可由**交叉验证**来选择最适合K值

  * 当K值较小的时候，表示使用较小领域的样本进行预测，因此会导致模型更加复杂，导致过拟合；
  * 当K值较大的时候，表示使用较大领域的样本进行预测，训练误差会增大，模型会简化，容易导致欠拟合

* **距离的度量：**一般使用欧式距离；

  * 欧式距离：若$a(a_1,a_2,a_3)$, $b(b_1,b_2,b_3)$，则两者的欧式距离为：
    $$
    \sqrt{(a1-b1)^2+(a2-b2)^2+(a2-b2)^2}
    $$
    

* **决策规则：**在*分类模型*中，主要使用多数表决或者加权多数表决法；在*回归模型*中，主要使用平均值法或者加权平均值法

  * 多数表决/均值法：每个邻近样本权重相同；
  * 加权多数表决/加权平均值法：每个邻近样本权重不同；一般情况下，采用权重和距离成反比的方式进行计算

### KNN算法实现

**蛮力实现(brute)**：

* 计算预测样本到所有训练集样本的距离，然后选择最小的k个距离即可得到K个最邻近点。 
* 缺点：计算消耗资源大

**KD树(kd tree)**：

* 对训练数据进行建模，构建KD树；
* 根据构建好的模型对样本进行预测；

> 除此之外，还有一些从KD树改进而来的求解最近邻点的算法，例如Ball Tree、BBF Tree、MVP Tree

### KD树浅析

当样本数量较少时，可以通过brute蛮力来求解最近邻；而当样本量较大的时候，KD树就能发挥其优势。

#### 构建方式

* 从m个样本的n维特征中，分别计算n个特征取值的方差；
* 用方差最大的第k维特征$n_k$作为根节点；
* 对于这个特征，选择取值的中位数$n_{kv}$作为样本的划分点，对于小于该值的样本划分到左子树，对于大于等于该值的样本划分到右子树；
* 对左右子树采用同样的方式找方差最大的特征作为根节点，递归即可产生KD树

#### 查找方式

* 对于一个目标点，首先在KD树里面找到包含目标点的叶子节点；
  * 从根节点出发，根据之前划分的条件，递归的向下访问KD树，直到达到叶子节点为止；
* 以目标点为圆心，以目标点到叶子节点样本实例的距离为半径，得出一个超球体，最近邻的点一定在这个超球体的内部；
* 返回到叶子节点的父节点，检查另一个子节点包含的超矩形区域是否和上述的超球体相交：
  * 若相交，则去这个子节点寻找是否有更加近的点，若有，则更新最近点；
* 若不相交，则继续回到叶子节点的父节点的父节点，在这个更父的父节点对应的另一个子树中继续上述步骤；
* 经过上述几步一直更新，当回溯到根节点时，最后的最近点就是当前目标点的最近邻点
* 把改点删除，继续进行上述的操作，直到找到K个点为止

> 下述博文中有关于此查找方式的案例，便于理解：
>
> https://cloud.tencent.com/developer/news/212042

### 实际应用：

* 示例代码

```python
from sklearn.datasets import load_iris
from sklearn.preprocessing import StandardScaler
from sklearn.neighbors import KNeighborsClassifier
from sklearn.model_selection import GridSearchCV

def knn_classifier_iris():
    """
    K-近邻预测鸢尾花
    """
    # 加载数据
    lr = load_iris()
    # 划分数据
    x_train, x_test, y_train, y_test = train_test_split(lr.data, lr.target, test_size=0.25)
    # 特征工程（标准化）
    std = StandardScaler()
    # 对测试集和训练集的特征值进行标准化
    x_train = std.fit_transform(x_train)
    x_test = std.transform(x_test)
    # 采用knn
    knn = KNeighborsClassifier(n_neighbors=3)
    # 训练
    # knn.fit(x_train, y_train)

    # # 得出预测
    # y_predict = knn.predict(x_test)
    # print(y_predict)

    # #评估模型
    # print("预测的准确率：", knn.score(x_test, y_test))
    # print("每个类别的精确率与召回率与F1Score", classification_report(y_test, y_predict, target_names=lr.target_names))

    # 采用网格搜索+交叉验证
    # 构造超参数的选择
    param = {"n_neighbors":[1,3,5]}
    
    # 构造网格搜索对象  2折交叉验证
    gc = GridSearchCV(knn, param_grid=param, cv=2)
    # 拟合
    gc.fit(x_train, y_train)
    # 预测+模型评估
    print("在测试集上的准确性：", gc.score(x_test, y_test))
    # 在测试集上的准确性： 0.9210526315789473
    print("在交叉验证当中的最好的结果：", gc.best_score_)
    # 在交叉验证当中的最好的结果： 0.9910714285714286
    print("最好的参数选择：", gc.best_params_)
    # 最好的参数选择： {'n_neighbors': 3}
    print("最好的模型：", gc.best_estimator_)
    # 最好的模型： KNeighborsClassifier(algorithm='auto', leaf_size=30, metric='minkowski', metric_params=None, n_jobs=1, n_neighbors=3, p=2, weights='uniform')
    print("每个超参数每次交叉验证的结果：", gc.cv_results_)
	# 略
    return None
```

> 由于这部分代码量太少，因此将其与决策树代码归到了一起，见下：https://github.com/zhuChengChao/ML-DecisionTree