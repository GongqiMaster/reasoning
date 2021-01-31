## LTP使用说明

官方附录，包含主要的词性标注集以及依存语法关系类型：

https://ltp.readthedocs.io/zh_CN/latest/appendix.html#id5

项目中对pyltp进行了二次封装，打包成了一个外部库ltp_parsing。mathnlp中的对应方法都是转调ltp_parsing的这层封装。

主要使用了ltp_parsing中的以下两个方法。

### Base LTP segment util

python版本源码位置：ltp_parsing/modules/segmentor.py

LTP提供了调用分词模块的接口，接受一个string作为输入，返回一个list作为输出（C++则是一个vector<string>）。

这部分的一个关键是加入了自定义词表。目前项目采用是”中学数学.txt.utf8“

候选词典：

- 数学.txt.utf8
- 数学词汇(融合了搜狗所有的数学词库).txt.utf8
- 一般数学.txt.utf8
- 中学数学.txt.utf8
- 数学词库.txt.utf8
- 数学词汇.txt.utf8
- 基础数学用词.txt.utf8
- 高中数学大词库.txt.utf8
- 数学词汇大全【官方推荐】.txt.utf8

### Base LTP tag util

python版本源码位置：ltp_parsing/modules/pos_tagger.py

LTP提供了调用词性模块的接口，接受一个list（vector<string>）作为输入，返回对应长度的词性list（vector<string>）。

这部分可以直接对接上面的分词模块的输出，同样，也可以对上面的分词结果魔改后再送入该模块。

**”处理——魔改——再处理“是能够得到较为精确地结果的理论基础（编的）。**

### 基于LTP dependency parse的关系抽取

此部分已被废弃。 

有时间会记录下思路。

### 备注

根据base segment util和base tag util得到结果必然是不够精确的、不能被人为控制的。因此分别要对接以下魔改部分：

- (mapped sentence) -> base segment -> max segment -> min segment -> (final words)
- (final words) -> synonym -> (final words)
- (final words) -> base tag -> repair tag -> (final tags)

