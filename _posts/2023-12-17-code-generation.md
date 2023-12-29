---
title: 代码生成（Code Generation）
date: 2023-12-17
categories: olap
---

> 这个是我在 [`DataFunConf2023-深圳站`](https://www.bagevent.com/event/8519252)上的分享整理的文字内容。

## 代码生成

代码生成（Code Generation）在数据库中有着广泛的应用。代码生成能将表达式，查询，计算过程等在程序执行的时候编译成机器码再执行，从而提高程序运行的效率。特别是随着硬件的发展，IO不再成为查询的瓶颈，对于计算密集型的查询，代码生成能极大提高查询性能。
![Hardware](/assets/images/jit/hardware.png)

## 为什么需要代码生成
我们以一个简单的查询来看一下数据库的处理流程：
```
SELECT
  price * 0.8 + distance / 1000 AS credit
FROM
  trips
WHERE
  arrival_city = 'Shenzhen’ AND duration < 120
```
![Query Plan](/assets/images/jit/query_plan.png)

这里主要看`Filter`阶段，也就是`WHERE`后面的表达式处理流程。通常情况下该表达式会被翻译成一个抽象语法书（`AST`）。

### 解释执行
我们通过一段简单的`Python`代码来看下解释执行的过程是处理的。

首先定义一个简单的类来表示`AST`对应的节点：
```python
class Node:
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right
```

对应上图中的`AST`，我们得到一个表达式树：
```python
expr_tree = Node(
    'AND',
    Node('=', Node('arrival_city'), Node('"Shenzhen"')),
    Node('<', Node('duration'), Node('120'))
)
```

采用深度优先（DFS）的方式遍历表达式树，对表达式求值。这里只处理了上述表达式中存在的几种情况。
- 调用根节点`AND`的`visit`函数，再分别调用左右子节点的`visit`函数，返回`AND`操作结果
- 调用左节点`=`的`visit`函数，比较字段`arrival_ciy`和字符串常量`Shenzhen`的值
- 调用又节点`<`的`visit`函数，比较字段`duration`和整型常量`120`的值

```python
def visit(node, record):
    if node is None:
        return None

    if node.value == 'AND':
        left_result = visit(node.left, record)
        right_result = visit(node.right, record)
        return left_result and right_result if left_result is not None and right_result is not None else None
    elif node.value == '<':
        left_value = get_value(node.left, record)
        right_value = get_value(node.right, record)
        return left_value < right_value if left_value is not None and right_value is not None else None
    elif node.value == '=':
        left_value = get_value(node.left, record)
        right_value = get_value(node.right, record)
        return left_value == right_value
    # 处理其他操作符和操作数的情况
    else:
        return None

def get_value(node, record):
    if node.value.isdigit():
        return int(node.value)
    elif node.value.startswith('"') and node.value.endswith('"'):
        return node.value[1:-1]
    elif node.value in record:
        return record[node.value]
    else:
        return None
```

这种解释执行的方式简单易理解，但是存在一些问题：
- 大量虚函数调用
  - 非确定性跳转指令，CPU无法做分支预测，打断CPU流水线
- 计算中无法确定类型，算子中存在很多动态类型判断
- 递归函数调用打断计算过程

## 什么是代码生成
代码生成就是生成需要执行的代码。这里我们还是使用`Python`来模拟采用代码生成的方式来处理上面的表达式：
```python
def generate_code(node):
    if node is None:
        return ""

    if node.value in ('AND', 'OR'):
        left_code = generate_code(node.left)
        right_code = generate_code(node.right)
        return f"({left_code}) {node.value.lower()} ({right_code})"
    elif node.value in ('>', '<'):
        return f"record['{node.left.value}'] {node.value} {node.right.value}"
    elif node.value == '=':
        return f"record['{node.left.value}'] == {node.right.value}"
    else:
        raise ValueError(f"Unsupported operation: {node.value}")

def jit_evaluator(expr_code, record):
    try:
        local_vars = {}
        exec(f"result = {expr_code}", {'record': record}, local_vars)
        return local_vars.get('result', None)
    except Exception as e:
        print(f"Error evaluating expression: {e}")
        return False

# 生成代码并执行
expr_code = generate_code(expr_tree)
print(f"Generated expression code: {expr_code}")

for record in database:
    result = jit_evaluator(expr_code, record)
    if result:
        print(f"Record {record} matches the filter.")
```
通常情况下我们会采用LLVM的中间语言（IR）作为生成的代码语言。整个表达式求值的执行过程就分为两步：
- 生成中间代码（IR）
- 编译器编译成机器码

其中生成中间代码通常会先进行**类型推导**，然后再生成代码。进行**类型推导**的过程也是检查表达式是否合法的过程。

编译器对生成的代码编译，得到机器可以执行的机器码，这里面就包含我们需要执行的类和函数。在执行过程中调用具体的逻辑就可以得到表达式的值。





  