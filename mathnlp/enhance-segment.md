## 分词增强方案

使用Base LTP得到分词结果后，会出现某些分词错误的情况：

- `等腰,直角,三角形`，应为`等腰直角三角形`
- `已知数列`，应为`已知,数列`

主要关心的是一些中文实体名分词错误的问题，因此有以下两步魔改处理。注意下面两步的顺序不能更改。

python版本源码位置：mathnerdevelop/utils/ltp_util.py

### 最大分词匹配模块

这一部分是将第一种中文实体不应被分割，但被分割的情况整合起来，用到的是分词中常用的”正向最大匹配“思路。

因此需要维护一张最大匹配分词表。之后再遇到某些词被分开的情况只需要在该表中添加即可。

最大分词匹配表位置：mathnerdevelop/map_files/max_match.txt

建议用数据库替换。

### 最小分词匹配模块

该部分是为了处理第二种中文实体应该被分割，但是没有被分割出来的情况。

同样需要维护一张最小匹配分词表，该表的格式为：

```shell
# 最小匹配词典，将分词结果不正确的词拆开
# "解是"->"解"和"是"
已知数列 已知|数列
有点 有|点
已知点 已知|点
```

以空格作为分隔，左边是原始的错误的情况，右边以`|`作为分隔符拆分左边的词语。目前只支持拆分成两个。

最小分词匹配表位置：mathnerdevelop/map_files/min_match.txt

建议用数据库替换。

### TRICK

将上面两种情况”善加利用“，便可以处理更加以外的情况。比如有

```
已,知数列
```

首先先用最大匹配模块整合，将`已知数列`添加到最大分词表中，再用最小分词模块将其拆开即可。

### 备注

经过这两部分后便能够得到最终的分词结果，并用于下一步LTP词性标注。

这两步魔改理论上可以应对绝大多数情况，目前还没有出现过不能适配的问题。若真的出现仍可在该位置添加新的魔改模块。