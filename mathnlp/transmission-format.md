## JSON 传输格式

本文规定了前后端交互的 JSON 格式。

### 输入 -> 前端

旧版前端只接受单一的句子输入，不支持题型和答案的输入：

```json
{
    "sentence": "some words"
}
```

新版前端输入需要支持题型、题干、小问、选择题选项等部分的输入：

```json
{
    "text_json":{
        "type":0,
        "stem":"qstem",
        "subStems":[
            "qsubstem1",
            "qsubstem2",
            "..."
        ],
        "options":[
            "choiceA",
            "choiceB",
            "choiceC",
            "choiceD"
        ],
        "solutions":[
            "subStem1Solution",
            "subStem2Solution",
            "..."
        ]
    },
    "chinese_type":"1"
}
```

字段说明：

- `text_json`：一道题目完整的所有输入信息
- `type`：输入的类型，字段类型为整型，目前支持结论`0`、选择题`1`、填空题`2`和大题`3`；
- `stem`：题干部分，字段类型为字符串，这里遵循如下输入规则：
	- `type == 0`，需要将所有结论部分写在`stem`中；**实例化的句子type都要设为0，不然无法正确获得isConclusion**；
	- `type == 1`，将选择题题干部分中**所有非问题部分**写在`stem`中，将问题部分写在`subStems`中；
	- `type == 2`，将填空题中的**所有非问题部分**写在`stem`中，将问题部分写在`subStems`中；
	- `type == 3`，将大题题干写在`stem`中。若该题存在小问，则将小问部分写在`subStems`中；若没有小问，问题在题干中，则题干只输入**不含问题的部分**，并将问题部分写在`subStems`中；
- `subStems`：小问部分，字段类型为字符串数组，主要遵循的规则如上所说。需要注意的有以下几点：
	- 若`type == 0`，则置空；
	- 选择题的答案不算小问，只有问题部分才算；
	- **最重要的一点**，若`type == 1`，则小问部分可以带有括号（中英文均可，但括号内部最好不能有除了空格以外其他的字符）；若`type == 2`，则小问部分可以带有连续的下划线；
- `options`：选择题答案部分，字段类型为字符串数组，只有`type == 1`时才会用到。这部分将会原封不动传送给后端；
- `solutions`：各个小问的解答过程；
- `chinese_type`：返回实体和关系的类型是否为中文，可以不传，只有在传这个参数且为1的情况下返回中文类型，不传或者传输别的值都返回英文类型；

### 前端 -> 后端

旧版前后端交互格式如下

```json
{
    "asking":"an entity",
    "name":"no use",
    "questionId":"unique id",
    "commonText":"all sentence",
    "fakeText":"sentence with introduced entities",
    "entities":[
        {
            "name":"A",
            "types":[
                "Point"
            ]
        },
        {
            "name":"ABC",
            "types":[
                "Triangle"
            ]
        }
    ],
    "relations":[
        {
            "entity1":"e1",
            "entity2":"e2",
            "type1":"t1",
            "type2":"t2",
            "relationString":"relation",
            "properties":[
                {
                    "propertyName":"P",
                    "propertyType":"Point"
                },
                {
                    "propertyName":"AB",
                    "propertyType":"Line"
                }
            ],
            "isQuestion":"question type",
            "isConclusion":"T or F"
        }
    ]
}
```

上面的格式可能产生的问题：

- `asking`只能问单个实体，既不能解决多个问题的情况，也不能问关系（问关系等同于问三元组）；
- 同一实体多个类型不好在`entities`中表示。

为了和新版输入 JSON 相匹配，暂定以下新版输出 JSON 格式：

```json
{
    "type":0,
    "questionId":"unique id",
    "commonText":"all sentence",
    "fakeText":"sentence with introduced entities",
    "entities":[
        {
            "name":"A",
            "types":[
                "Point"
            ]
        },
        {
            "name":"ABC",
            "types":[
                "Triangle"
            ]
        }
    ],
    "stemRelations":[
        {
            "entity1":"e1",
            "entity2":"e2",
            "type1":"t1",
            "type2":"t2",
            "relationString":"relation",
            "properties":[
                {
                    "propertyName":"P",
                    "propertyType":"Point"
                },
                {
                    "propertyName":"AB",
                    "propertyType":"Line"
                }
            ],
            "isConclusion":0
        }
    ],
    "subStemRelations":[
        {
            "isQuestion":"question type",
            "relations":[
                {
                    "entity1":"e1",
                    "entity2":"e2",
                    "type1":"t1",
                    "type2":"t2",
                    "relationString":"relation",
                    "properties":[
                        {
                            "propertyName":"P",
                            "propertyType":"Point"
                        },
                        {
                            "propertyName":"AB",
                            "propertyType":"Line"
                        }
                    ],
                    "asking":0
                }
            ]
        },
        {
            "isQuestion":"question type",
            "relations":[
                {
                    "entity1":"e1",
                    "entity2":"e2",
                    "type1":"t1",
                    "type2":"t2",
                    "relationString":"relation",
                    "properties":[
                        {
                            "propertyName":"P",
                            "propertyType":"Point"
                        },
                        {
                            "propertyName":"AB",
                            "propertyType":"Line"
                        }
                    ],
                    "asking":0
                }
            ]
        }
    ],
    "options":[
        "choiceA",
        "choiceB",
        "choiceC",
        "choiceD"
    ],
    "solutionRelations":[
        [
            {
                "entity1":"e1",
                "entity2":"e2",
                "type1":"t1",
                "type2":"t2",
                "relationString":"relation",
                "properties":[
                    {
                        "propertyName":"P",
                        "propertyType":"Point"
                    }
                ],
                "isConclusion":0
            }
        ],
        [
            {
                "entity1":"e1",
                "entity2":"e2",
                "type1":"t1",
                "type2":"t2",
                "relationString":"relation",
                "properties":[
                    {
                        "propertyName":"P",
                        "propertyType":"Point"
                    }
                ],
                "isConclusion":0
            }
        ]
    ]
}
```

主要有以下改动：

- `type`返回对应的输入的类型；
- 若`type == 1`，将选择题答案`options`原封不动返回，其他情况置空；
- 原有的三元组`relations`部分被分解为题干部分的三元组`stemRelations`和小问部分的三元组`subStemRelations`。其中`subStemRelations`是一个二维的三元组对象数组，为了兼容多个小问的情况，每一维是一个小问的三元组对象数组；
- 对于每个小问，有各自的`isQuestion`字段，标明该小问的题型，暂定以下枚举类型：
	- `0`：求解型问题
	- `1`：证明型问题
	- `2`：判断型问题
	- `3`：选择题的证明小问（选择题将选项接入小问中当做证明题）
- 对于题干三元组和解题过程三元组当中的`isConclusion`字段，定义以下枚举类型：
  - `0`：非结论性、非条件性普通陈述句子；
  - `1`：条件关系；
  - `2`：结论关系。
- 对于特殊的解题字段，`stemRelations`中只包含`isConclusion`，且当`type == 0`时所有的都是`F`；`subStemRelations`中只包含`isQuestion`和一个新增的`asking`字段，这是一个枚举的整型类型，定义了以下9种问题:
	- `0`：不是问题三元组，没有要求的；
	- `1`：求`entity1`；
	- `2`：求`entity2`；
	- `3`：求`relationString`；
	- `4`：求`entity1`和`entity2`；
	- `5`：求`entity1`和`relationString`；
	- `6`：求`entity2`和`relationString`；
	- `7`：求`entity1`、`entity2`和`relationString`。
	- `8`： 求该关系下的所有属性

上面格式可能产生的问题：

- 实例化类型没有区分，这样在求解不同的题目的时候，都会将实例化都遍历一遍，这样效率比较低，出错率也比较高。
- 在同类型实例化下，有可能会有好多实例化，并且在求解是调用实例化也有顺序要求，需要手动指定实例化的顺序，这样不便自动调题。
- 实例化内容比较多，可能不方便查看，因此需要简短的实例化描述。

为了适应新的需求，暂定 json 格式如下：

```json
{
    "type":0,
    "instantiatedLevel":0,
    "instantiatedCategory":"instantiated with category",
    "instantiatedDescription":"instantiated description",
    "questionId":"unique id",
    "commonText":"all sentence",
    "fakeText":"sentence with introduced entities",
    "entities":[
        {
            "name":"A",
            "types":[
                "Point"
            ]
        },
        {
            "name":"ABC",
            "types":[
                "Triangle"
            ]
        }
    ],
    "stemRelations":[
        {
            "entity1":"e1",
            "entity2":"e2",
            "type1":"t1",
            "type2":"t2",
            "relationString":"relation",
            "properties":[
                {
                    "propertyName":"P",
                    "propertyType":"Point"
                },
                {
                    "propertyName":"AB",
                    "propertyType":"Line"
                }
            ],
            "isConclusion":0
        }
    ],
    "subStemRelations":[
        {
            "isQuestion":"question type",
            "relations":[
                {
                    "entity1":"e1",
                    "entity2":"e2",
                    "type1":"t1",
                    "type2":"t2",
                    "relationString":"relation",
                    "properties":[
                        {
                            "propertyName":"P",
                            "propertyType":"Point"
                        },
                        {
                            "propertyName":"AB",
                            "propertyType":"Line"
                        }
                    ],
                    "asking":0
                }
            ]
        },
        {
            "isQuestion":"question type",
            "relations":[
                {
                    "entity1":"e1",
                    "entity2":"e2",
                    "type1":"t1",
                    "type2":"t2",
                    "relationString":"relation",
                    "properties":[
                        {
                            "propertyName":"P",
                            "propertyType":"Point"
                        },
                        {
                            "propertyName":"AB",
                            "propertyType":"Line"
                        }
                    ],
                    "asking":0
                }
            ]
        }
    ],
    "options":[
        "choiceA",
        "choiceB",
        "choiceC",
        "choiceD"
    ],
    "solutionRelations":[
        [
            {
                "entity1":"e1",
                "entity2":"e2",
                "type1":"t1",
                "type2":"t2",
                "relationString":"relation",
                "properties":[
                    {
                        "propertyName":"P",
                        "propertyType":"Point"
                    }
                ],
                "isConclusion":0
            }
        ],
        [
            {
                "entity1":"e1",
                "entity2":"e2",
                "type1":"t1",
                "type2":"t2",
                "relationString":"relation",
                "properties":[
                    {
                        "propertyName":"P",
                        "propertyType":"Point"
                    }
                ],
                "isConclusion":0
            }
        ]
    ]
}
```

主要改动如下：

只有type=0 的情况下，才返回下面字段

- `instantiatedLevel ` ：返回实例化的优先级，数字越大优先级越高，定义0-1000
- `instantiatedCategory` ： 返回实例化的类别数，比如公用0，函数1，向量2，数列3，解析几何4，复数5，平面几何6，立体几何7
- `instantiatedDescription` ：返回实例化的简短描述，如函数求导，等差数列前n项和

