---
layout: post
title:  "机器学习 - Neural Network"
date:   2019-01-02 16:46:01 +0800
categories: machine-learning
---

## 读取数据

```python
import pandas as pd
import numpy as np
train_data  = pd.read_csv('handwritten_digits.csv',header=None)

index = list(range(0,400))
X = train_data.as_matrix(index);
y = train_data[400];
```

## 数据可视化

```python
from matplotlib import pyplot as plt
def displayData(X,row=None,col=None):
    m,n=X.shape
    w = h = int(np.sqrt(n))
    if row==None or col == None:
        row = col = int(np.sqrt(m))
    data = []
    index = 0
    for r in range(0,row):
        rows = []
        for c in range(0,col):
            rows.append(np.reshape(X[index,:],(w,h),order='F'))
            index = index + 1
        data.append(rows)
    
    rows = []
    for r in range(0,row):
        t = np.concatenate(tuple(data[r]),axis=1);
        rows.append(t);
    
    c = np.concatenate(tuple(rows));
    plt.imshow(c,cmap="gray")
    plt.axis('off')
    plt.show();
        
m,n=X.shape
rand_indice = np.random.permutation(m)
displayData(X[rand_indice[:49]]);
```

![png]({{ "assets/images/machine-learning-ex3/neural_network/output_1_0.png" | relative_url}})

## 模型训练

```python
from sklearn.neural_network import MLPClassifier
model = MLPClassifier(solver='lbfgs', alpha=1e-5,hidden_layer_sizes=(10,), random_state=1);
clf = model.fit(X,y)
print("Training Set Accuracy: ",clf.score(X,y))
```

> Training Set Accuracy:  0.969

## 模型预测

```python
m,n=X.shape
rand_indice = np.random.permutation(m)
row=4
col=4
test_train = X[rand_indice[:row*col]]
displayData(test_train,row,col)
print(np.reshape(clf.predict(test_train),(row,col)))
```

![png]({{ "assets/images/machine-learning-ex3/neural_network/output_3_0.png" | relative_url}})

> [ [10 10  5  1]
>   [ 1  1  2  5]
>   [ 5  1  7  9]
>   [ 7  9  8  1] ]

## 参考

* [Neural Networks Supervised - sklearn][1]

[1]:https://scikit-learn.org/stable/modules/neural_networks_supervised.html "Neural Networks Supervised - sklearn"
