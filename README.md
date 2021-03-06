
> 最近碰到一位兄台的毕设题目是: 基于用户画像的音乐推荐系统, 然后这哥们就被网上的各种资料给绕进去了, 比如item2vec, 各种基于模型的, 偏偏是跟用户画像没关系, 为了应付过去可以按照以下思路来做
> 更多资料关注公众号 【推荐系统与机器学习】

## 准备数据

### 正排数据, 直接扔redis里, 可以直接通过id查询正排信息

```protobuf

message Pair {
    string tag;
    float weight;
}

message Music {
    
    string id;
    string title;
    string author;
    repeated Pair tags; //歌曲的tag列表, 比如电子/时尚/爱情等标签, 标签有多个 并且有对应权重， 最大权重为1
}
```


### 画像定义

```protobuf

messsage UserProfile {
    string uid; //用户ID
    
    repeated Pair tags; //用户的标签体系, 与正排对应
    repeated Pair authors; //用户的喜欢音乐人, 由于是多个, 所以是列表, 与正排对应
}
```



### 埋点数据

```javascript
{
    'id' : '12412414', //音乐music id,
    'uid' : '2412125125', //用户id
    'avg' : 0.5 //播放时长百分比, 用于做用户对改首音乐的喜爱权重
}
```


### 画像构建


每次处理点击时, 拿到该用户的过去某一段时间的所有点击

根据点击能够拿到点击对应的所有正排, 然后 tag 权重 * 点击数据权重，做聚合, 如果是author的话，可以认为权重为1

然后可以构造出该用户的这类数据

```text
author 周杰伦 2.5
author 蔡依林 1.2
author 五月天 3.4

tag 爱情 4.5
tag 民谣 2.3
tag 摇滚 5.6
```

然后设计一个sigmoid函数（自己随便设计）, 主要是让 author/tag 对应的权重在 0-1 区间之内, 然后这个就是你的画像



### 开始做推荐

有了用户画像之后, 基于正排做倒排链, 正排是 id->整个item属性, 倒排就是 属性->id列表 （其实就是mysql一张表, id, tag, weight/id,author,weight, 可以根据tag按照weight降序查该tag有哪些歌曲）

比如你要候选集生成100个。。然后按照画像里的权重, 可能需要 周杰伦拿出10个， 蔡依林拿出5个， 爱情拿出20个。。

拿到这100个后, 按照上面的那俩protobuf 做cos相似度。。直接前几选出来, 更高级的做法可以使用LR+GBDT等算法进行选择

为了演示个性化, 需要保证刷两次之后， 该音乐不再出来


### 进阶

通过在产品设计上加入歌单功能, 可以让用户帮你准备数据, 基于歌单列表, 可以做一个item2vec 的 映射关系, 这个在候选集生成的时候可以用到

