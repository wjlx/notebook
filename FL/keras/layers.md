# 层

[TOC]

## 关于 Keras 网络层

所有 Keras 网络层都有很多共同的函数：

- `layer.get_weights()`: 以含有Numpy矩阵的列表形式返回层的权重。
- `layer.set_weights(weights)`: 从含有Numpy矩阵的列表中设置层的权重（与get_weights的输出形状相同）。
- `layer.get_config()`: 返回包含层配置的字典。此图层可以通过以下方式重置：

```python
layer = Dense(32)
config = layer.get_config()
reconstructed_layer = Dense.from_config(config)
```

或

```python
from keras import layers

config = layer.get_config()
layer = layers.deserialize({'class_name': layer.__class__.__name__,
                            'config': config})
```

如果一个层具有单个节点 (i.e. 如果它不是共享层), 你可以得到它的输入张量、输出张量、输入尺寸和输出尺寸:

- `layer.input`
- `layer.output`
- `layer.input_shape`
- `layer.output_shape`

如果层有多个节点 (参见: <font color=red size=3>层节点和共享层的概念</font>), 您可以使用以下函数:

- `layer.get_input_at(node_index)`
- `layer.get_output_at(node_index)`
- `layer.get_input_shape_at(node_index)`
- `layer.get_output_shape_at(node_index)`

## 基础层

### keras.engine.base_layer.Layer

### keras.engine.base_layer.InputSpec

### keras.engine.base_layer.Node

## 核心网络层

## 卷积层 Convolutional

## 池化层 Pooling

## 局部连接层 Locally-connected

## 循环层 Recurrent

## 嵌入层 Embedding

## 融合层 Merge

## 高级激活层 Advanced Activations

## 标准化层 Normalization

## 噪声层 Noise

## 层封装器 Wrappers

## 编写你自己的层

## keras.layers.BatchNormalization