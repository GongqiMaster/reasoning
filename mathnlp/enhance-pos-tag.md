## 词性修复模块

Base LTP tag util接受的是经过两步分词魔改后的结果和同义词替换后的最终分词结果。并生成对应长度的词性标注list（vector<string>）。

python版本源码位置：mathnerdevelop/utils/ltp_util.py

### 实体在词性上的表征

实体包含两部分：

- 中文实体类型
- 英文实体名称

对于英文实体名称，此时已被替换为单个的大写英文字符，base LTP tag util为其分配的词性标签一定是`ws`。

对于中文实体类型，我们规定只有名词才能作为实体，即词性标签为`n`或以`n`开头。而事实上也是如此，绝大多数实体就是名词，这也符合一般的NER任务中对实体的定义。

### 词性修复

那么对于LTP标错词性的情况，就需要进行一些手工的处理。因为我们只关心实体的词性被打错标签的情况，所以只需要建立一张词性修复表，修复那些出现过的被打错词性标签的中文类型即可：

```shell
# 登记某些词性标注出错的词，修复词性
# word origin_tag|origin_tag|...|repair_tag
向量 p|n
圆心 a|n
点 v|q|n
集合 v|n
等腰直角三角形 v|n
圆 a|v|n
对边 nd|n
```

以空格分界，左边是词性被标记错误的实体，右边是应调整的标签。规定右边这样编写：以`|`分界，左侧所有的错误词性情况都应被替换为最右侧的词性。

词性修复表位置：mathnerdevelop/map_files/tag_repair.txt

建议用数据库替换。

### 备注

注意，以`oe`开头的英文实体，会有可能出现词性不是`ws`的现象，主要有两种解决方案：
1. 将该实体添加到词性修复表中；
2. 取消对英文实体判断时的词性要求。

目前采用的第一种。

经过原始的LTP词性标注，以及下一步的词性修复，能够应对大部分词性标注错误的情况。经过分词和词性标注两步得到的两个等长的list，会被用于接下来的数学命名实体识别任务。