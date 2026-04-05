神经网络 - 架构师学习笔记
    
    


    
        
            
                神经网络
            
            
            
                
                    神经网络是深度学习的基础，模拟人脑神经元的工作方式来处理信息。它在图像识别、自然语言处理等领域取得了突破性进展。
                
                
                
                    
                        理解神经网络的原理和实现对于开发智能应用和设计AI系统架构至关重要。
                    
                
                
                神经网络基础概念
                
                
                    
                        
                            神经元(Neuron)
                        
                        
                            神经网络的基本单元，接收输入信号，经过加权求和和激活函数处理后输出结果。
                        
                    
                    
                    
                        
                            层(Layer)
                        
                        
                            由多个神经元组成的集合，常见的有输入层、隐藏层和输出层。
                        
                    
                    
                    
                        
                            权重和偏置
                        
                        
                            权重决定输入信号的重要性，偏置提供额外的灵活性来拟合数据。
                        
                    
                    
                    
                        
                            激活函数
                        
                        
                            引入非线性因素，使神经网络能够学习复杂的模式。
                        
                    
                
                
                神经网络工作原理
                
                
                    
                        前向传播
                        
import numpy as np

# 简单的神经网络前向传播示例
def sigmoid(x):
    return 1 / (1 + np.exp(-x))

# 输入数据
X = np.array([[0.5, 0.3]])

# 权重和偏置
W1 = np.array([[0.2, 0.4], [0.1, 0.3]])
b1 = np.array([[0.1, 0.2]])
W2 = np.array([[0.3], [0.2]])
b2 = np.array([[0.1]])

# 前向传播
z1 = np.dot(X, W1) + b1
a1 = sigmoid(z1)
z2 = np.dot(a1, W2) + b2
a2 = sigmoid(z2)

print(f"输入: {X}")
print(f"第一层输出: {a1}")
print(f"最终输出: {a2}")
                        
                            前向传播是数据从输入层经过隐藏层到输出层的流动过程。
                        
                    
                    
                    
                        反向传播
                        
# 反向传播计算梯度示例
def sigmoid_derivative(x):
    return x * (1 - x)

# 假设真实标签
y_true = np.array([[1]])

# 计算损失
loss = 0.5 * (y_true - a2)**2

# 反向传播
dL_da2 = a2 - y_true
da2_dz2 = sigmoid_derivative(a2)
dL_dz2 = dL_da2 * da2_dz2

dL_dW2 = np.dot(a1.T, dL_dz2)
dL_db2 = dL_dz2

dL_da1 = np.dot(dL_dz2, W2.T)
da1_dz1 = sigmoid_derivative(a1)
dL_dz1 = dL_da1 * da1_dz1

dL_dW1 = np.dot(X.T, dL_dz1)
dL_db1 = dL_dz1

print(f"损失: {loss}")
print(f"W1梯度: {dL_dW1}")
print(f"W2梯度: {dL_dW2}")
                        
                            反向传播通过链式法则计算损失函数对各参数的梯度。
                        
                    
                
                
                常用激活函数
                
                
                    
                        Sigmoid
                        
def sigmoid(x):
    return 1 / (1 + np.exp(-x))

def sigmoid_derivative(x):
    return x * (1 - x)
                        
                            将值映射到(0,1)区间，常用于二分类问题的输出层。
                        
                    
                    
                    
                        ReLU
                        
def relu(x):
    return np.maximum(0, x)

def relu_derivative(x):
    return (x > 0).astype(float)
                        
                            现在最常用的激活函数，计算简单且能缓解梯度消失问题。
                        
                    
                    
                    
                        Tanh
                        
def tanh(x):
    return np.tanh(x)

def tanh_derivative(x):
    return 1 - np.tanh(x)**2
                        
                            将值映射到(-1,1)区间，输出以0为中心。
                        
                    
                    
                    
                        Softmax
                        
def softmax(x):
    exp_x = np.exp(x - np.max(x))  # 防止溢出
    return exp_x / np.sum(exp_x)
                        
                            常用于多分类问题的输出层，使输出表示概率分布。
                        
                    
                
                
                深度学习框架
                
                
                    
                        TensorFlow/Keras示例
                        
import tensorflow as tf
from tensorflow import keras

# 创建简单的神经网络
model = keras.Sequential([
    keras.layers.Dense(128, activation='relu', input_shape=(784,)),
    keras.layers.Dropout(0.2),
    keras.layers.Dense(10, activation='softmax')
])

# 编译模型
model.compile(optimizer='adam',
              loss='sparse_categorical_crossentropy',
              metrics=['accuracy'])

# 显示模型结构
model.summary()
                        
                            TensorFlow是Google开发的开源机器学习框架，Keras是其高级API。
                        
                    
                    
                    
                        PyTorch示例
                        
import torch
import torch.nn as nn
import torch.optim as optim

# 定义神经网络
class Net(nn.Module):
    def __init__(self):
        super(Net, self).__init__()
        self.fc1 = nn.Linear(784, 128)
        self.dropout = nn.Dropout(0.2)
        self.fc2 = nn.Linear(128, 10)
        
    def forward(self, x):
        x = torch.relu(self.fc1(x))
        x = self.dropout(x)
        x = self.fc2(x)
        return x

# 创建模型实例
model = Net()

# 定义损失函数和优化器
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters())
                        
                            PyTorch是Facebook开发的开源深度学习框架，具有动态计算图的特点。
                        
                    
                
                
                常见网络架构
                
                
                    
                        
                            卷积神经网络(CNN)
                        
                        
                            主要用于图像处理，通过卷积层提取空间特征。
                        
                    
                    
                    
                        
                            循环神经网络(RNN)
                        
                        
                            适用于序列数据处理，如文本和时间序列，具有记忆能力。
                        
                    
                    
                    
                        
                            Transformer
                        
                        
                            基于注意力机制的架构，在自然语言处理领域表现卓越。
                        
                    
                    
                    
                        
                            生成对抗网络(GAN)
                        
                        
                            由生成器和判别器组成，用于生成新的数据样本。
                        
                    
                
                
                
                    神经网络训练技巧
                    
                        合理初始化权重，避免梯度消失或爆炸
                        使用批量归一化加速训练
                        采用合适的优化算法，如Adam、RMSprop
                        使用学习率调度策略
                        实施正则化技术防止过拟合
