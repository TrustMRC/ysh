# ysh
## 基于擦除的阅读理解可解释性分析

1、使用jieba分词对句子进行分词（去除标点），得到一个ner_list，遍历ner_list中的每个词语，擦除变成[MASK]，计算模型输出得到的end_logits/ start_logits和不加擦除的原句输出得到的end_logits/ start_logits之间的均方误差值作为loss，同样的词若出现多次，则取多次的MSE平均值作为最终的loss。
```
举例:
'context'=地瓜不是红薯。地瓜一般生吃或者凉拌，外形是纺锤型的，有明显的瓣状结构，内里的肉是白色的，有清淡的药香味，生吃又脆又甜，常食用可以预防肝癌、胃癌，营养价值非常高。红薯是粗粮，也叫番薯山芋。它是一种属管状花目，旋花科一年生的草本植物，富含丰富的矿物质和维生素，而且非常耐饱。地瓜是红薯吗'
mse_loss=[('是', 17911358.315734863), ('红薯', 184779.06158447266), ('地瓜', 178871.92962646484), ('生', 17667.507934570312), ('吃', 17667.507934570312), ('有', 17667.507934570312), ('粗粮', 2196.8502807617188), ('叫', 2196.8502807617188), ('番薯', 2196.8502807617188), ('山芋', 2196.8502807617188), ('属', 2196.8502807617188), ('管状花', 2196.8502807617188), ('旋', 2196.8502807617188), ('富含', 2196.8502807617188), ('矿物质', 2196.8502807617188), ('凉拌', 1606.1370849609375), ('外形', 1606.1370849609375), ('纺锤', 1606.1370849609375), ('瓣状', 1606.1370849609375), ('结构', 1606.1370849609375)]
```
2、接下来是对词性的判断，如果该词是名词或动词，loss乘以系数10（经实验证明10对提升F1效果最好）。
3、对loss按照从大到小进行排序，loss越大意味着这个词对答案提取的影响程度越大，即该词更有可能作为证据词，取topk个作为证据词。
4、loss可以和att或者IG分数结合进一步优化。
