# Server端的计算图

(简体中文|[English](./SERVER_DAG.md))

本文档显示了Server端上计算图的概念。 如何使用PaddleServing内置运算符定义计算图。 还显示了一些顺序执行逻辑的示例。

## Server端的计算图

深度神经网络通常在输入数据上有一些预处理步骤，而在模型推断分数上有一些后处理步骤。 由于深度学习框架现在非常灵活，因此可以在训练计算图之外进行预处理和后处理。 如果要在服务器端进行输入数据预处理和推理结果后处理，则必须在服务器上添加相应的计算逻辑。 此外，如果用户想在多个模型上使用相同的输入进行推理，则最好的方法是在仅提供一个客户端请求的情况下在服务器端同时进行推理，这样我们可以节省一些网络计算开销。 由于以上两个原因，自然而然地将有向无环图（DAG）视为服务器推理的主要计算方法。 DAG的一个示例如下：

<center>
<img src='server_dag.png' width = "450" height = "500" align="middle"/>
</center>

## 如何定义节点

PaddleServing在框架中具有一些预定义的计算节点。 一种非常常用的计算图是简单的reader-infer-response模式，可以涵盖大多数单一模型推理方案。 示例图和相应的DAG定义代码如下。
<center>
<img src='simple_dag.png' width = "260" height = "370" align="middle"/>
</center>

``` python
import paddle_serving_server as serving
op_maker = serving.OpMaker()
read_op = op_maker.create('general_reader')
general_infer_op = op_maker.create('general_infer')
general_response_op = op_maker.create('general_response')

op_seq_maker = serving.OpSeqMaker()
op_seq_maker.add_op(read_op)
op_seq_maker.add_op(general_infer_op)
op_seq_maker.add_op(general_response_op)
```

由于该代码在大多数情况下都会被使用，并且用户不必更改代码，因此PaddleServing会发布一个易于使用的启动命令来启动服务。 示例如下：

``` python
python -m paddle_serving_server.serve --model uci_housing_model --thread 10 --port 9292
```

## 更多示例

如果用户将稀疏特征作为输入，并且模型将对每个特征进行嵌入查找，则我们可以进行分布式嵌入查找操作，该操作不在Paddle训练计算图中。 示例如下：

``` python
import paddle_serving_server as serving
op_maker = serving.OpMaker()
read_op = op_maker.create('general_reader')
dist_kv_op = op_maker.create('general_dist_kv')
general_infer_op = op_maker.create('general_infer')
general_response_op = op_maker.create('general_response')

op_seq_maker = serving.OpSeqMaker()
op_seq_maker.add_op(read_op)
op_seq_maker.add_op(dist_kv_op)
op_seq_maker.add_op(general_infer_op)
op_seq_maker.add_op(general_response_op)
```
