# 2017知乎看山杯机器学习挑战赛 

## Koala队伍解决方案

### 运行环境

基于Python2及PyTorch，需安装：
1. PyTorch
2. Numpy
3. visdom
4. fire

运行：
```shell
cd src; pip2 install -r requirements.txt
python2 -m visdom.server
```

### 数据分析

代码在data_analysis文件夹下

### 数据处理

代码在data_preprocess文件夹下，question_preprocess、label_preprocess、topic_preprocess，分别有对应的notebook和py版本。

### 单模型训练

代码在src文件夹下，需在其中新建snapshots文件夹，用于存储模型文件

- dataset：存放数据load文件
- models：存放所有模型定义文件，主要用到FastText.py、TextCNN.py、RNN.py
- utils：存放工具文件，如模型加载与保存、日志、可视化、矩阵处理等
- config.py：配置文件，可在运行时通过命令行修改
- main.py：所有程序入口

在src下运行：
```shell
python2 main.py train --model=RNN --use_word=True --batch_size=256
```

上述命令后面都是可设置的参数
- model是使用的模型（与models下文件名一致，结果保存在snapshots/模型名/）
- use_word表示使用word训练，如使用char，则改为--use_char=True
- batch_size表示训练batch，显存不够的可以适当减小
- 还有其他一些参数，见config.py中的配置

### Boosting模型训练

对于单个模型来说，其所能实现的效果毕竟有限。通过分析数据，我们发现一个模型对于不同类别是具有偏向性的，即有的类可能会全部预测错，而另一个类则会全部预测对，这种类别之间的差异性对预测性能会有很大的影响
因此，我们针对这种偏差，借鉴Boost提升的思想，提出了一个新颖的做法，对结果进行修复性训练多层并累加。

在src下运行：
```shell
python2 main.py train --model=RNN --use_word=True --batch_size=256　--boost=True --base_layer=0
```

将base_layer依次改为1、２、３...，可逐层训练，训练的累加结果保存在与模型同目录

### 各模型结果

**线下结果，线上可高2个多千分点**

word结果

单模型:
- FastText: 0.4097
- TextCNN: 0.4111
- RNN: 0.4116

Boosting模型:
- FastText10层: 0.41892
- RNN10层: 0.42642
- TextCNN10层: 0.42654

char结果比word低约1个百分点，但融合后会涨3个千分点左右

### 测试

- 加载训好的模型并测试：参考gen_test_res.py
- 直接融合各模型测试的结果文件：参考utils/resmat.py

### 细节

- 更详细的描述，请移步知乎解决方案文章：https://zhuanlan.zhihu.com/p/29020616
