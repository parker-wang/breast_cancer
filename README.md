# breast_cancer

## 乳腺癌检测:
采用SVM方法，对美国威斯康星州的乳腺癌诊断数据集进行分类，最终实现一个针对乳腺癌检测的分类器 

数据集来自美国威斯康星州的乳腺癌诊断数据集
医疗人员采集了患者乳腺肿块经过细针穿刺 (FNA) 后的数字化图像，并且对这些数字图像进行了特征提取，这些特征可以描述图像中的细胞核呈现。肿瘤可以分成良性和恶性。部分数据截屏如下所示：

![image](https://github.com/mrtungleung/breast_cancer/blob/master/images/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-09-10%20%E4%B8%8A%E5%8D%8810.18.44.png)

数据表一共包括了 32 个字段，代表的含义如下：

![image](https://github.com/mrtungleung/breast_cancer/blob/master/images/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-09-10%20%E4%B8%8A%E5%8D%8810.35.03.png)

上面的表格中，mean 代表平均值，se 代表标准差，worst 代表最大值（3 个最大值的平均值）。每张图像都计算了相应的特征，得出了这 30 个特征值（不包括 ID 字段和分类标识结果字段 diagnosis），实际上是 10 个特征值（radius、texture、perimeter、area、smoothness、compactness、concavity、concave points、symmetry 和fractal_dimension_mean）的 3 个维度，平均、标准差和最大值。这些特征值都保留了4 位数字。字段中没有缺失的值。在 569 个患者中，一共有 357 个是良性，212 个是恶性。

好了，我们的目标是生成一个乳腺癌诊断的 SVM 分类器，并计算这个分类器的准确率。
首先设定项目的执行流程：

![image](https://github.com/mrtungleung/breast_cancer/blob/master/images/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-09-10%20%E4%B8%8A%E5%8D%8810.44.37.png)

1. 首先我们需要加载数据源；2. 在准备阶段，需要对加载的数据源进行探索，查看样本特征和特征值，这个过程你也可以使用数据可视化，它可以方便我们对数据及数据之间的关系进一步加深了解。然后按照“完全合一”的准则来评估数据的质量，如果数据质量不高就需要做数据清洗。数据清洗之后，你可以做特征选择，方便后续的模型训练；3. 在分类阶段，选择核函数进行训练，如果不知道数据是否为线性，可以考虑使用SVC(kernel=‘rbf’) ，也就是高斯核函数的 SVM 分类器。然后对训练好的模型用测试集进行评估。
按照上面的流程，我们来编写下代码，加载数据并对数据做部分的探索：

![image](https://github.com/mrtungleung/breast_cancer/blob/master/images/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-09-10%20%E4%B8%8A%E5%8D%8810.45.55.png)

这是部分的运行结果。

![image](https://github.com/mrtungleung/breast_cancer/blob/master/images/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-09-10%20%E4%B8%8A%E5%8D%8810.46.44.png)

接下来，我们就要对数据进行清洗了。
运行结果中，你能看到 32 个字段里，id 是没有实际含义的，可以去掉。diagnosis 字段的取值为 B 或者 M，我们可以用 0 和 1 来替代。另外其余的 30 个字段，其实可以分成三组字段，下划线后面的 mean、se 和 worst 代表了每组字段不同的度量方式，分别是平均值、标准差和最大值。

![image](https://github.com/mrtungleung/breast_cancer/blob/master/images/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-09-10%20%E4%B8%8A%E5%8D%8810.47.15.png)

然后我们要做特征字段的筛选，首先需要观察下 features_mean 各变量之间的关系，这里我们可以用 DataFrame 的 corr() 函数，然后用热力图帮我们可视化呈现。同样，我们也会看整体良性、恶性肿瘤的诊断情况。

![image](https://github.com/mrtungleung/breast_cancer/blob/master/images/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-09-10%20%E4%B8%8A%E5%8D%8810.47.56.png)

这是运行的结果：

![image](https://github.com/mrtungleung/breast_cancer/blob/master/images/1.png)
![image](https://github.com/mrtungleung/breast_cancer/blob/master/images/201600150008.png)

热力图中对角线上的为单变量自身的相关系数是 1。颜色越浅代表相关性越大。所以你能看出来 radius_mean、perimeter_mean 和 area_mean 相关性非常大，compactness_mean、concavity_mean、concave_points_mean 这三个字段也是相关的，因此我们可以取其中的一个作为代表。
那么如何进行特征选择呢？特征选择的目的是降维，用少量的特征代表数据的特性，这样也可以增强分类器的泛化能力，避免数据过拟合。
我们能看到 mean、se 和 worst 这三组特征是对同一组内容的不同度量方式，我们可以保留 mean 这组特征，在特征选择中忽略掉 se 和 worst。同时我们能看到 mean 这组特征中，radius_mean、perimeter_mean、area_mean 这三个属性相关性大，compactness_mean、daconcavity_mean、concave points_mean 这三个属性相关性大。我们分别从这 2 类中选择 1 个属性作为代表，比如 radius_mean 和compactness_mean。
这样我们就可以把原来的 10 个属性缩减为 6 个属性，代码如下：

![image](https://github.com/mrtungleung/breast_cancer/blob/master/images/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-09-10%20%E4%B8%8A%E5%8D%8810.49.28.png)

对特征进行选择之后，我们就可以准备训练集和测试集：

![image](https://github.com/mrtungleung/breast_cancer/blob/master/images/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-09-10%20%E4%B8%8A%E5%8D%8810.49.47.png)

在训练之前，我们需要对数据进行规范化，这样让数据同在同一个量级上，避免因为维度问题造成数据误差：

![image](https://github.com/mrtungleung/breast_cancer/blob/master/images/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-09-10%20%E4%B8%8A%E5%8D%8810.50.20.png)

最后我们可以让 SVM 做训练和预测了：

![image](https://github.com/mrtungleung/breast_cancer/blob/master/images/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-09-10%20%E4%B8%8A%E5%8D%8810.50.37.png)

运行结果：

![image](https://github.com/mrtungleung/breast_cancer/blob/master/images/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-09-10%20%E4%B8%8A%E5%8D%8810.50.57.png)

准确率大于 90%，说明训练结果还不错。
