# 通用
1. [PaddleNLP常见问题汇总（持续更新）](https://paddlenlp.readthedocs.io/zh/latest/FAQ.html)
2. 待学习 [PaddleNLP教程文档](https://blog.csdn.net/qq_56591814/article/details/128468604) | [PaddleNLP教程文档](https://zhuanlan.zhihu.com/p/639275106)
3. 待实践 [PaddleNLP一键预测功能：Taskflow API](https://paddlenlp.readthedocs.io/zh/latest/model_zoo/taskflow.html) | [PaddleNLP 一键预测功能 Taskflow API 使用教程](https://aistudio.baidu.com/aistudio/projectdetail/3702870)
4. [Github PaddleNLP](https://github.com/PaddlePaddle/PaddleNLP)

# 环境安装
```python
pip install paddlepaddle paddlepaddle-gpu
pip install paddlenlp
```

# 预训练模型
1. [PaddleNLP Transformer预训练模型](https://paddlenlp.readthedocs.io/zh/latest/model_zoo/index.html)

## 样本需求
很难定义具体需要多少条样本，取决于具体的任务以及数据的质量。如果数据质量没问题的话，分类、文本匹配任务所需数据量级在百级别，翻译则需要百万级能够训练出一个比较鲁棒的模型。如果样本量较少，可以考虑数据增强，或小样本学习。

# 数据集
## 加载本地数据集
通过使用PaddleNLP提供的 load_dataset， MapDataset 和 IterDataset ，可以方便的自定义属于自己的数据集。

```python
from paddlenlp.datasets import load_dataset

def read(data_path):
    with open(data_path, 'r', encoding='utf-8') as f:
        # 跳过列名
        next(f)
        for line in f:
            words, labels = line.strip('\n').split('\t')
            words = words.split('\002')
            labels = labels.split('\002')
            yield {'tokens': words, 'labels': labels}

# data_path为read()方法的参数
map_ds = load_dataset(read, data_path='train.txt', lazy=False)
iter_ds = load_dataset(read, data_path='train.txt', lazy=True)
```

## 加载预定义数据集
```python
import paddlenlp as ppnlp
from paddlenlp.datasets import load_dataset

train_ds, dev_ds, test_ds = load_dataset(
    "chnsenticorp", splits=["train", "dev", "test"])

print(train_ds.label_list)

for data in train_ds.data[:5]:
    print(data)
```

## 数据处理
1. [基于预训练模型的数据处理](https://paddlenlp.readthedocs.io/zh/latest/data_prepare/data_preprocess.html#id2)
2. [基于非预训练模型的数据处理](https://paddlenlp.readthedocs.io/zh/latest/data_prepare/data_preprocess.html#id3)

## tokenizer

### 其他
- [JiebaTokenizer](https://paddlenlp.readthedocs.io/zh/latest/source/paddlenlp.data.tokenizer.html)
- [提速不掉点：基于词颗粒度的中文WoBERT](https://kexue.fm/archives/7758/comment-page-1#Tokenizer)

## 数据增强
PaddleNLP提供了Data Augmentation[数据增强API](https://github.com/PaddlePaddle/PaddleNLP/blob/develop/docs/dataaug.md "数据增强API")，可用于训练数据数据增强

其他：
- [文本中的数据增强](https://zhuanlan.zhihu.com/p/420565311)

# 模型导入保存
## PaddleNLP预训练模型
保存
```python
model.save_pretrained('./checkpoint')
tokenizer.save_pretrained('./checkpoint')
```
加载
```python
model.from_pretrained("./checkpoint')
tokenizer.from_pretrained("./checkpoint')
```

## 自定义预训练模型
```python
# 加载
tokenizer = BertTokenizer.from_pretrained("bert-base-uncased")
model = BertModel.from_pretrained("bert-base-uncased")
# 调用save_pretrained()生成 model_config.json、 tokenizer_config.json、model_state.pdparams、 vocab.txt 文件，保存到./checkpoint
tokenizer.save_pretrained("./checkpoint")
model.save_pretrained("./checkpoint")
# 修改model_config.json，tokenizer_config.json这两个配置文件，指定为自己的模型，之后通过from_pretrained()加载模型
tokenizer = BertTokenizer.from_pretrained("./checkpoint")
model = BertModel.from_pretrained("./checkpoint")
```

# 模型训练
1. [PaddleNLP Trainer API](https://paddlenlp.readthedocs.io/zh/latest/trainer.html)

## 继续热启动训练
先将lr、 optimizer、model的参数保存下来
```python
paddle.save(lr_scheduler.state_dict(), "xxx_lr")
paddle.save(optimizer.state_dict(), "xxx_opt")
paddle.save(model.state_dict(), "xxx_para")
```
加载lr、 optimizer、model参数即可恢复训练
```python
lr_scheduler.set_state_dict(paddle.load("xxxx_lr"))
optimizer.set_state_dict(paddle.load("xxx_opt"))
model.set_state_dict(paddle.load("xxx_para"))
```

## 冻结模型梯度
冻结训练其实也是迁移学习的思想，在目标检测任务中用得十分广泛。因为目标检测模型里，主干特征提取部分所提取到的特征是通用的，把backbone冻结起来训练可以加快训练效率，也可以防止权值被破坏。在冻结阶段，模型的主干被冻结了，特征提取网络不发生改变，占用的显存较小，仅对网络进行微调。在解冻阶段，模型的主干不被冻结了，特征提取网络会发生改变，占用的显存较大，网络所有的参数都会发生改变。

参考：
1. [Pytorch在训练时冻结某些层使其不参与训练](https://blog.csdn.net/qq_36429555/article/details/118547133)
2. [如何冻结模型梯度](https://paddlenlp.readthedocs.io/zh/latest/FAQ.html#q3-4)


## 小样本
增加训练样本带来的效果是最直接的。此外，可以基于我们开源的预训练模型进行热启，再用少量数据集fine-tune模型。此外，针对分类、匹配等场景，[小样本学习](https://github.com/PaddlePaddle/PaddleNLP/tree/develop/examples/few_shot "小样本学习")也能够带来不错的效果。

## 并行
1. [关于生成式语言大模型的一些工程思考 paddlenlp & chatglm](https://juejin.cn/post/7250291603813285946)

## 其他
1. [如何在eval阶段打印评价指标，在各epoch保存模型参数？](https://paddlenlp.readthedocs.io/zh/latest/FAQ.html#q3-5-eval-epoch)

# 模型输出

# 推理服务部署

## 预测部署
在动态图模式下开发，静态图模式部署。

### 动转静
动转静，即将动态图的模型转为可用于部署的静态图模型。 动态图接口更加易用，python 风格的交互式编程体验，对于模型开发更为友好，而静态图相比于动态图在性能方面有更绝对的优势。因此动转静提供了这样的桥梁，同时兼顾开发成本和性能。 可以参考官方文档 [动态图转静态图文档](https://www.paddlepaddle.org.cn/documentation/docs/zh/develop/guides/04_dygraph_to_static/index_cn.html "动态图转静态图文档")，使用 `paddle.jit.to_static` 完成动转静。 另外，在 PaddleNLP 我们也提供了导出静态图模型的例子，可以参考 [waybill_ie 模型导出](https://github.com/PaddlePaddle/PaddleNLP/tree/develop/examples/information_extraction/waybill_ie/#%E6%A8%A1%E5%9E%8B%E5%AF%BC%E5%87%BA "waybill_ie 模型导出")。

### 借助Paddle Inference部署
动转静之后保存下来的模型可以借助Paddle Inference完成高性能推理部署。Paddle Inference内置高性能的CPU/GPU Kernel，结合细粒度OP横向纵向融合等策略，并集成 TensorRT 实现模型推理的性能提升。具体可以参考文档 [Paddle Inference 简介](https://paddleinference.paddlepaddle.org.cn/master/product_introduction/inference_intro.html "Paddle Inference 简介")。 为便于初次上手的用户更易理解 NLP 模型如何使用Paddle Inference，PaddleNLP 也提供了对应的例子以供参考，可以参考 [/PaddleNLP/examples](https://github.com/PaddlePaddle/PaddleNLP/tree/develop/examples/ "/PaddleNLP/examples") 下的deploy目录，如[基于ERNIE的命名实体识别模型部署](https://github.com/PaddlePaddle/PaddleNLP/tree/develop/examples/information_extraction/waybill_ie/deploy/python "基于ERNIE的命名实体识别模型部署")。

## 提升QPS
从工程角度，对于服务器端部署可以使用 [Paddle Inference](https://www.paddlepaddle.org.cn/documentation/docs/zh/guides/05_inference_deployment/inference/inference_cn.html "Paddle Inference") 高性能预测引擎进行预测部署。对于Transformer类模型的GPU预测还可以使用PaddleNLP中提供的[FastGeneration](https://github.com/PaddlePaddle/PaddleNLP/tree/develop/paddlenlp/ops "FastGeneration")功能来进行快速预测，其集成了[NV FasterTransformer](https://github.com/NVIDIA/FasterTransformer "NV FasterTransformer")并进行了功能增强。

从模型策略角度，可以使用一些模型小型化技术来进行模型压缩，如模型蒸馏和裁剪，通过小模型来实现加速。PaddleNLP中集成了ERNIE-Tiny这样一些通用小模型供下游任务微调使用。另外PaddleNLP提供了[模型压缩示例](https://github.com/PaddlePaddle/PaddleNLP/tree/develop/examples/model_compression "模型压缩示例")，实现了DynaBERT、TinyBERT、MiniLM等方法策略，可以参考对自己的模型进行蒸馏压缩。

参考：
1. [文本生成高性能加速](https://github.com/PaddlePaddle/PaddleNLP/blob/develop/paddlenlp/ops/README.md)

# 案例
## 文本分类
1. [中文新闻文本标题分类](https://openi.pcl.ac.cn/xiaoxiong/Classification-of-Chinese-news-text-headings)
2. [PaddleNLP基于ERNIR3.0文本分类以中医疗搜索检索词意图分类(KUAKE-QIC)为例【多分类(单标签)】](https://www.ctfiot.com/76474.html)

## 实体抽取
1. [医疗领域实体抽取：UIE Slim最新升级版含数据标注、serving部署、模型蒸馏等教学，助力工业应用场景快速落地](https://aijishu.com/a/1060000000405173)
2. [军事领域关系抽取：UIE Slim最新升级版含数据标注、serving部署、模型蒸馏等教学，助力工业应用场景快速落地](https://aijishu.com/a/1060000000405517)
3. [【Paddle打比赛】基于PaddleNLP的面向低资源和增量类型的命名实体识别](https://blog.csdn.net/m0_63642362/article/details/131476360)

## 信息抽取
1. PDF [基于ERNIELayout&pdfplumber-UIE的多方案学术论文信息抽取](https://www.jianshu.com/p/aa02295afcc4)
2. 重要 [文本抽取任务UIE Taskflow使用指南](https://github.com/PaddlePaddle/PaddleNLP/blob/develop/applications/information_extraction/taskflow_text.md)
3. [产业实践分享：基于UIE-X的医疗文档信息提取，少样本微调大幅提升抽取效果](https://aistudio.baidu.com/aistudio/projectdetail/5261592)
4. [简历信息提取（七）：用ERNIE-Layout实现文档智能问答信息筛选](https://blog.csdn.net/m0_63642362/article/details/129226494)

## 文本生成
1. [文本生成任务实战：如何使用PaddleNLP实现各种解码策略](https://aistudio.baidu.com/aistudio/projectdetail/3243711)

## 垂直领域
1. [算法框架-预训练-增量/全量自定义垂直领域预训练模型](https://zhuanlan.zhihu.com/p/465462642)

## 增量学习
1. [增量学习-学习总结（上）](https://zhuanlan.zhihu.com/p/384273982)
1. [增量学习(Incremental Learning)小综述](https://www.scholat.com/teamwork/showPostMessage.html?id=9840)

# UIE
[信息抽取应用](https://github.com/PaddlePaddle/PaddleNLP/tree/develop/applications/information_extraction/#readme) | 
1. 论文：[Unified Structure Generation for Universal Information Extraction](https://aclanthology.org/2022.acl-long.395.pdf)
2. [PaddleNLP之UIE信息抽取小样本进阶(二)[含doccano详解]](https://aistudio.baidu.com/aistudio/projectdetail/4160689?contributionType=1)
3. [信息抽取UIE（二）--小样本快速提升性能（含doccona标注](https://developer.aliyun.com/article/1064871)
4. [通用信息抽取技术UIE产业案例解析，Prompt范式落地经验分享！](https://www.paddlepaddle.org.cn/support/news?action=detail&id=3323)
5. [基于Label studio实现UIE信息抽取智能标注方案，提升标注效率！](https://aistudio.baidu.com/aistudio/projectdetail/5770279?contributionType=1&sUid=691158&shared=1&ts=1685327308357)
6. [UIE与ERNIE-Layout：智能视频问答任务初探](https://aistudio.baidu.com/aistudio/projectdetail/5439436)
7. [跨模态文档通用信息抽取模型UIE-X来了](https://aistudio.baidu.com/aistudio/projectdetail/5017442)

# ERNIE
- [ERNIE: Enhanced Representation through kNowledge IntEgration](https://github.com/PaddlePaddle/PaddleNLP/tree/develop/model_zoo/ernie-1.0/ "ERNIE: Enhanced Representation through kNowledge IntEgration")

# 杂项
## pipreqs 生成依赖
```
# 生成requirements.txt
pipreqs ./ --encoding=utf8 --proxy http://${host}:${port}

# 自动安装所有依赖
pip install -r requriements.txt
```
参考：
- https://pypi.org/project/pipreqs/

## partial

## stream
- [Map, Filter, Reduce – Working on Streams in Python](https://learnpython.com/blog/map-filter-reduce-python/)

## tqdm / pbar

## 其他
```python
# 多行if
if cond1 == 'val1' and \
   cond2 == 'val2' and \
   cond3 == 'val3' and \
   cond4 == 'val4':
    do_something
```
