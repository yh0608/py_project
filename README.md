# 电影推荐系统python实现
>寒假在家保持写代码不手生，实现了这个小推荐系统。

简介：推荐系统的一种简单实现就是，给定一个用户A，找到所有用户中与A最相似的用户B，把B看过的电影中A没看过的挑出来，再把B评分最高的几部挑出来。
# 1 数据说明
数据集下载ml-latest-small(1MB): [http://files.grouplens.org/datasets/movielens/ml-latest-small.zip](http://files.grouplens.org/datasets/movielens/ml-latest-small.zip)

解压缩后用到两个文件 **movies.csv** 和 **ratings.csv** 。
**movies.csv**是各种电影的数据，列分别为 电影编号、电影名、所属类型。

| movieId | title | genres |
|--- | --- | --- | 
1  |Toy Story (1995) | Adventure,Animation,Children,Comedy,Fantasy
2   | Jumanji (1995)  |    Adventure,Children,Fantasy

**ratings.csv**用户的评分数据，列分别为 用户编号、电影编号、评分、时间戳。

|userId | movieId | rating | timestamp|
| --- | ---- | ---- | ---|
1  |1   |  4.0 | 964982703
1  |3   | 4.0  |964981247
# 2 数据处理
我们的目的是给定一个用户id，找出他可能喜欢的电影名。
但是两个文件电影信息和用户评分信息是分开的，所以需要合并。
## 2.1读取原始数据
```
import pandas as pd
movies = pd.read_csv(r'C:\Users\yyy\Desktop\推荐系统\movies.csv') #注意含中文路径需要在前面加 r 转义
ratings = pd.read_csv(r'C:\Users\yyy\Desktop\推荐系统\ratings.csv')
```
## 2.2合并两个文件
```
data = pd.merge(movies,ratings,on = 'movieId')#通过两数据框之间的movieId连接
data[['userId','rating','movieId','title']].sort_values('userId').to_csv(r'C:\Users\yyy\Desktop\推荐系统\merged.csv',index=False)
```
## 2.3 用字典存放所得数据
```
file = open(r'C:\Users\yyy\Desktop\推荐系统\merged.csv','r')#记得读取文件时加‘r’， encoding='UTF-8'
##读取data.csv中每行中除了名字的数据
data = {}##存放每位用户评论的电影和评分
for line in file.readlines():
    #注意这里不是readline()
    line = line.strip().split(',')
    #如果字典中没有某位用户，则使用用户ID来创建这位用户
    if not line[0] in data.keys():
        data[line[0]] = {line[3]:line[1]}
    #否则直接添加以该用户ID为key字典中
    else:
        data[line[0]][line[3]] = line[1]
```
此时得到的`data[:2]`

|movieId      |       title    |   genres  |userId|rating|timestamp|
|--- | --- | --- | --- | --- | --- |
1 | Toy Story (1995) | Adventure,Animation,Children,Comedy,Fantasy    |1| 4.0 | 964982703
1 | Toy Story (1995) | Adventure,Animation,Children,Comedy,Fantasy   | 5 |4.0 |  847434962 
# 3 推荐系统
## 3.1 计算两个用户的相似度
**注意：最后把距离缩放到了[0, 1]之间，这是为了简化计算。因为有可能两个用户之间的差异很大，平方和累加起来是一个很大的数，他们两个差异这么大对这个推荐系统没用，所以用1/（1+distance）把它缩放到0.**
```
from math import pow, sqrt
def Euclidean(user1,user2):
    #取出两位用户评论过的电影和评分
    user1_data=data[user1]
    user2_data=data[user2]
    distance = 0
    #找到两位用户都评论过的电影，并计算欧式距离
    for key in user1_data.keys():
        if key in user2_data.keys():
            #注意，distance越大表示两者越相似
            distance += pow(float(user1_data[key])-float(user2_data[key]),2)
 
    return 1/(1+sqrt(distance))#这里返回值越大，相似度越大
```
## 3.2 找到最相似的k个用户
```
def top10_similar(userID):
    res = []
    for userid in data.keys():
        if not userid == userID:
            sim = Euclidean(userID, userid)
            res.append((userid, sim))
    res.sort(key=lambda val:val[1], reverse=True)
    
    return res[:10]
    
RES = top10_similar('1')
print(RES)
```
## 3.3 找到最相似的用户看过的电影
```
def recommend(user, k=5):
    recomm = []
    most_sim_user = top10_similar(user)[0][0]
    items = data[most_sim_user]
    for item in items.keys():
        if item not in data[user].keys():
            recomm.append((item, items[item]))
    recomm.sort(key=lambda val:val[1], reverse=True)
    
    return recomm[:k]
        
RECOM = recommend('1')
print(RECOM)
```
