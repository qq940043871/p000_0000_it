自然语言处理 - 架构师学习笔记
    
    


    
        
            
                自然语言处理
            
            
            
                
                    自然语言处理(NLP)是人工智能的一个重要分支，致力于让计算机理解和生成人类语言，实现人机自然交流。
                
                
                
                    
                        随着深度学习和预训练模型的发展，NLP技术在机器翻译、文本摘要、情感分析等领域取得了显著进展。
                    
                
                
                NLP核心任务
                
                
                    
                        
                            文本分类
                        
                        
                            将文本分配到预定义的类别中，如垃圾邮件检测、情感分析等。
                        
                    
                    
                    
                        
                            命名实体识别
                        
                        
                            识别文本中的特定实体，如人名、地名、机构名等。
                        
                    
                    
                    
                        
                            机器翻译
                        
                        
                            将一种语言的文本自动翻译成另一种语言。
                        
                    
                    
                    
                        
                            问答系统
                        
                        
                            根据用户问题从文档或知识库中找到答案。
                        
                    
                
                
                文本预处理技术
                
                
                    
                        分词
                        
import jieba
import nltk
from nltk.tokenize import word_tokenize

# 中文分词
chinese_text = "自然语言处理是人工智能的重要分支"
chinese_tokens = jieba.lcut(chinese_text)
print("中文分词:", chinese_tokens)

# 英文分词
english_text = "Natural language processing is an important branch of AI"
english_tokens = word_tokenize(english_text)
print("英文分词:", english_tokens)
                        
                            分词是NLP的第一步，将连续的文本切分成有意义的词汇单元。
                        
                    
                    
                    
                        词干提取和词形还原
                        
from nltk.stem import PorterStemmer, WordNetLemmatizer

# 词干提取
stemmer = PorterStemmer()
words = ["running", "ran", "runs", "easily", "fairly"]
stems = [stemmer.stem(word) for word in words]
print("词干提取:", stems)

# 词形还原
lemmatizer = WordNetLemmatizer()
lemmas = [lemmatizer.lemmatize(word) for word in words]
print("词形还原:", lemmas)
                        
                            词干提取和词形还原用于将词汇变体归一化为基本形式。
                        
                    
                    
                    
                        停用词过滤
                        
from nltk.corpus import stopwords

# 获取停用词列表
stop_words = set(stopwords.words('english'))
print("英文停用词:", list(stop_words)[:10])

# 过滤停用词
text = "this is a sample sentence showing off the stop words filtration"
words = word_tokenize(text)
filtered_words = [word for word in words if word.lower() not in stop_words]
print("过滤停用词后:", filtered_words)
                        
                            停用词过滤可以去除对语义贡献较小的常见词汇。
                        
                    
                
                
                文本表示方法
                
                
                    
                        词袋模型(Bag of Words)
                        
from sklearn.feature_extraction.text import CountVectorizer

# 示例文本
corpus = [
    "This is the first document.",
    "This document is the second document.",
    "And this is the third one.",
    "Is this the first document?",
]

# 创建词袋模型
vectorizer = CountVectorizer()
X = vectorizer.fit_transform(corpus)

print("词汇表:", vectorizer.get_feature_names_out())
print("词频矩阵:")
print(X.toarray())
                        
                            词袋模型忽略词汇顺序，仅考虑词汇出现的频率。
                        
                    
                    
                    
                        TF-IDF
                        
from sklearn.feature_extraction.text import TfidfVectorizer

# 创建TF-IDF向量器
tfidf_vectorizer = TfidfVectorizer()
tfidf_matrix = tfidf_vectorizer.fit_transform(corpus)

print("TF-IDF特征名称:", tfidf_vectorizer.get_feature_names_out())
print("TF-IDF矩阵:")
print(tfidf_matrix.toarray())
                        
                            TF-IDF考虑了词频和逆文档频率，能够更好地表示词汇的重要性。
                        
                    
                    
                    
                        词嵌入(Word Embeddings)
                        
# 使用gensim加载预训练的Word2Vec模型
# from gensim.models import KeyedVectors
# model = KeyedVectors.load_word2vec_format('word2vec.bin', binary=True)

# 获取词向量
# vector = model['computer']
# print("computer的词向量:", vector)

# 计算词汇相似度
# similarity = model.similarity('computer', 'laptop')
# print("computer和laptop的相似度:", similarity)
                        
                            词嵌入将词汇映射到低维连续向量空间，能够捕捉词汇间的语义关系。
                        
                    
                
                
                NLP深度学习模型
                
                
                    
                        循环神经网络(RNN) for NLP
                        
import torch
import torch.nn as nn

class SimpleRNN(nn.Module):
    def __init__(self, vocab_size, embedding_dim, hidden_dim, output_dim):
        super(SimpleRNN, self).__init__()
        self.embedding = nn.Embedding(vocab_size, embedding_dim)
        self.rnn = nn.RNN(embedding_dim, hidden_dim, batch_first=True)
        self.fc = nn.Linear(hidden_dim, output_dim)
        
    def forward(self, x):
        embedded = self.embedding(x)
        output, hidden = self.rnn(embedded)
        # 取最后一个时间步的输出
        last_output = output[:, -1, :]
        return self.fc(last_output)

# 模型参数
vocab_size = 10000
embedding_dim = 128
hidden_dim = 256
output_dim = 2  # 二分类

# 创建模型
model = SimpleRNN(vocab_size, embedding_dim, hidden_dim, output_dim)
print(model)
                        
                            RNN能够处理变长序列数据，在NLP任务中广泛应用。
                        
                    
                    
                    
                        Transformer架构
                        
# Transformer的关键组件：自注意力机制
import torch
import torch.nn.functional as F

def scaled_dot_product_attention(Q, K, V, mask=None):
    # 计算注意力分数
    scores = torch.matmul(Q, K.transpose(-2, -1)) / torch.sqrt(torch.tensor(Q.shape[-1], dtype=torch.float32))
    
    # 应用掩码（如果提供）
    if mask is not None:
        scores = scores.masked_fill(mask == 0, -1e9)
    
    # 应用softmax获取注意力权重
    attention_weights = F.softmax(scores, dim=-1)
    
    # 计算加权和
    output = torch.matmul(attention_weights, V)
    
    return output, attention_weights

# 示例使用
Q = torch.randn(1, 3, 4)  # batch_size=1, seq_len=3, d_model=4
K = torch.randn(1, 3, 4)
V = torch.randn(1, 3, 4)

output, weights = scaled_dot_product_attention(Q, K, V)
print("输出形状:", output.shape)
print("注意力权重:", weights)
                        
                            Transformer基于自注意力机制，能够并行处理序列数据，在NLP领域表现卓越。
                        
                    
                
                
                预训练语言模型
                
                
                    
                        
                            BERT
                        
                        
                            双向Transformer编码器，通过掩码语言模型和下一句预测进行预训练。
                        
                    
                    
                    
                        
                            GPT
                        
                        
                            基于Transformer解码器的生成式预训练模型，擅长文本生成任务。
                        
                    
                    
                    
                        
                            T5
                        
                        
                            将所有NLP任务统一为文本到文本的转换任务。
                        
                    
                    
                    
                        
                            中文预训练模型
                        
                        
                            如BERT-wwm、RoBERTa-wwm、ERNIE等，专门针对中文文本进行优化。
                        
                    
                
                
                
                    NLP应用最佳实践
                    
                        根据任务特点选择合适的预训练模型
                        进行领域适应微调以提升性能
                        处理好数据不平衡问题
                        使用合适的评估指标
                        注意模型的可解释性和公平性
