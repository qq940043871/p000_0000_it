机器学习基础 - 架构师学习笔记
    
    


    
        
            
                机器学习基础
            
            
            
                
                    机器学习是人工智能的核心领域之一，它使计算机能够从数据中学习并做出预测或决策，而无需明确编程。
                
                
                
                    
                        理解机器学习的基本概念和算法对于架构师设计智能系统至关重要。
                    
                
                
                机器学习类型
                
                
                    
                        
                            监督学习
                        
                        
                            使用标记数据进行训练，学习输入和输出之间的映射关系。
                        
                        
                            分类
                            回归
                        
                    
                    
                    
                        
                            无监督学习
                        
                        
                            从未标记的数据中发现隐藏的模式和结构。
                        
                        
                            聚类
                            降维
                        
                    
                    
                    
                        
                            强化学习
                        
                        
                            通过与环境交互学习最优行为策略，以最大化累积奖励。
                        
                        
                            Q学习
                            策略梯度
                        
                    
                
                
                核心算法
                
                
                    
                        线性回归
                        
import numpy as np
from sklearn.linear_model import LinearRegression
import matplotlib.pyplot as plt

# 生成示例数据
X = np.array([[1], [2], [3], [4], [5]])
y = np.array([2, 4, 6, 8, 10])

# 创建并训练模型
model = LinearRegression()
model.fit(X, y)

# 预测
y_pred = model.predict(X)

# 可视化结果
plt.scatter(X, y, color='blue', label='实际值')
plt.plot(X, y_pred, color='red', label='预测值')
plt.legend()
plt.show()

print(f"斜率: {model.coef_[0]}")
print(f"截距: {model.intercept_}")
                        
                            线性回归用于建立输入变量和连续输出变量之间的线性关系。
                        
                    
                    
                    
                        决策树
                        
from sklearn.tree import DecisionTreeClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

# 示例数据
X = [[0, 0], [1, 1], [1, 0], [0, 1]]
y = [0, 1, 1, 0]

# 创建决策树分类器
clf = DecisionTreeClassifier()

# 训练模型
clf.fit(X, y)

# 预测
predictions = clf.predict([[2, 2], [1, 0]])

print(f"预测结果: {predictions}")
                        
                            决策树通过树状结构进行决策，易于理解和解释。
                        
                    
                    
                    
                        K均值聚类
                        
from sklearn.cluster import KMeans
import numpy as np
import matplotlib.pyplot as plt

# 生成示例数据
X = np.array([[1, 2], [1, 4], [1, 0],
              [10, 2], [10, 4], [10, 0]])

# 创建K均值聚类器
kmeans = KMeans(n_clusters=2, random_state=0)

# 训练模型
kmeans.fit(X)

# 预测聚类标签
labels = kmeans.labels_

# 获取聚类中心
centers = kmeans.cluster_centers_

print(f"聚类标签: {labels}")
print(f"聚类中心: {centers}")
                        
                            K均值聚类将数据分为K个簇，使同一簇内的数据点相似度高。
                        
                    
                
                
                机器学习流程
                
                
                    
                        1
                        
                            问题定义
                            明确要解决的问题类型（分类、回归、聚类等）。
                        
                    
                    
                    
                        2
                        
                            数据收集与预处理
                            收集相关数据并进行清洗、转换和特征工程。
                        
                    
                    
                    
                        3
                        
                            模型选择与训练
                            选择合适的算法并使用训练数据训练模型。
                        
                    
                    
                    
                        4
                        
                            模型评估
                            使用测试数据评估模型性能，调整参数。
                        
                    
                    
                    
                        5
                        
                            模型部署与监控
                            将模型部署到生产环境并持续监控性能。
                        
                    
                
                
                常用Python库
                
                
                    
                        
                            Scikit-learn
                        
                        
                            简单高效的数据挖掘和数据分析工具，包含大量机器学习算法。
                        
                    
                    
                    
                        
                            Pandas
                        
                        
                            提供高性能、易用的数据结构和数据分析工具。
                        
                    
                    
                    
                        
                            NumPy
                        
                        
                            Python科学计算的基础包，提供多维数组对象和各种派生对象。
                        
                    
                    
                    
                        
                            Matplotlib
                        
                        
                            Python 2D绘图库，用于创建高质量的图表和可视化。
                        
                    
                
                
                
                    机器学习最佳实践
                    
                        确保数据质量和数量充足
                        进行适当的特征工程
                        使用交叉验证评估模型性能
                        避免过拟合和欠拟合
                        持续监控和更新模型
