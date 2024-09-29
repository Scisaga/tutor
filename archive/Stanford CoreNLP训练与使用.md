## Stanford Segmenter
Stanford Segmenter把一段给定的文字分为单个的词。是tagging和tree-parsing的先决条件。

### 训练

训练一个segmenter主要需要三个文件，训练直接需要的文件都在`/train-material/`文件夹里
1. 一个分好词的语料。在本例中是`train-material/corpora.txt`，这个文件是从ctb8.0得到的。用`train.makeTrainFile`生成。
   - `\train-material\segmented`是ctb8.0里面的原始文件。需要去掉各种xml格式，每行一句，用空格隔开
2. 一个prop文件，用来指定各种相关参数，在本例中使用的是`train-material/ctb.prop`，这个prop文件是直接从stanford-segmenter-2015-04-20\data\ctb.prop抄过来的，其中大多数参数的功能未知
   - `trainFile` 参数可能是用来指定哪个文件是被训练的语料，但是没有它似乎也没有问题
3. 一个字典文件。这个文件似乎是不必要的，但是有更好一些。本例中用的字典是`\train-material\tfelab-dict.ser.gz`
   - 这个字典是把原来的字典`stanford-segmenter-2015-04-20\data\dict-chris6.ser`里面的词提取出来，加上所有我们手里的词典`\dict\dict-src\`生成的
   - `\dict\dict-chris-raw\`是所有`dict-chris6.ser`中的词，这些文件是通过getDict.DictUtil生成的
   - `\dict\dict-chris-src\`中是处理词典的中间结果
   - `\dict\dict-chris-src\chris-minus是在\dict\dict-src\` 中没有，`但是在dict-chris6.ser` 中有的词表
      - `chris-cleaned` 是minus的词中由纯汉字构成，而且不包含 把，被，使，一，年 等字的词。
      - `chris-filtered+chris-cleaned=chris-minus`
      - `chris-filtered-foreign-name` 是 `chris-filtered` 中所有外文人名，去除间隔号并打散的结果
      - `chris-filtered-others = chris-filtered - chris-filtered-foreign-name`
   - `\dict\dict-output`里面最终的结果，等于 `\dict\dict-src\ + chris-cleaned + chris-filtered-foreign-name` 是用类io.txt2gz生成的

在linux下执行训练的命令为：
```java
java -Xms128m -Xmx30g \
        -XX:MaxTenuringThreshold=0 \
        -classpath stanford-segmenter-3.6.0.jar:slf4j-simple.jar:slf4j-api.jar edu.stanford.nlp.ie.crf.CRFClassifier \
        -prop /opt/seg/train/ctb.prop \
        -serDictionary /opt/seg/train/dict-tfelab.ser.gz \
        -sighanCorporaDict data \
        -multiThreadGrad 16 \
        -multiThreadPerceptron 16 \
        -multiThreadGibbs 16 \
        -multiThreadClassifier 16 \
        -trainFile /opt/seg/train/ctb8.txt \
        -serializeTo /opt/newmodel.ser.gz
```

### Java 类
- `edu.stanford.nlp.ie.crf.CRFClassifier`
   - 在训练时，不仅需要上面三个文件，还需要 `stanford-segmenter-2015-04-20\data`，文件夹被复制到根目录中。
   - 参数说明：
      - "-prop","/opt/stanford-segmenter/train-material/ctb.prop", //指定prop文件的位置
      - "-serDictionary", "/opt/stanford-segmenter/train-material/dict-tfelab.ser.gz", //指定字典文件的位置。字典不是gz格式也没关系
      - "-sighanCorporaDict", "data", // 不知道什么用
      - "-multiThreadGrad", "20",  // 四个设置线程数的命令
      - "-multiThreadPerceptron", "20",
      - "-multiThreadGibbs", "20",
      - "-multiThreadClassifier", "20",
      - "-trainFile", "/opt/stanford-segmenter/train-material/corpora.txt",  //语料文件的位置
      - "-serializeTo", "newmodel.ser.gz", ">", "newmodel.log", "2>", "newmodel.err"; //输出文件的目录

注意：所有的路径都必须是全路径，否则系统不识别

### 使用
- 用`segmenter.Segmenter`类实现。输入未分词的句子，输出分好词的句子。主类：`edu.stanford.nlp.ie.crf.CRFClassifier`

## Stanford Tagger

### 训练

训练一个tagger主要需要文件：ctb8tagger.txt，这个文件是从ctb8.0得到的。用train.makeTrainFile生成，需要去掉原始文件中的xml格式。

trainCTB.java实现了tagger文件的生成：
   - "-model", "/opt/stanford-tagger/tfelab.tagger",
   - "-trainFile", "/opt/stanford-tagger/ctb8.txt",
   - "-arch","generic,suffix(4),prefix(4),unicodeshapes(-1,1),unicodeshapeconjunction(-1,1),words(-2,-2),words(2,2)",
   - "-tagSeparator","_",
   - "-nthreads","24"
   - `-model`指定生成的模型文件，`-trainFile` 指定从ctb8.0中的到的语料文件，-arch 不知道作用，-tagSeparator 指定分隔符。-nthreads 指定线程数。

训练完成生成tfelab.tagger文件，替换stanford原先的chinese-nodistsim.tagger文件。

文档中的路径都是在linux下的路径，训练的时候根据实际情况替换路径参数

训练的具体实现类详见：/tfelab-tag/blob/master/src/main/java/org/tfelab/tag/train/trainCTB.java

## Stanford DependencyParser
### 训练
训练一个DependencyParser主要需要三个文件：
- 原始语料
   - 将ctb8.0\data\bracketed文件夹中的文件合并生成 `ctb8tree.txt`'''`，具体生成参见 `MakeTreeFile.java`
- CoNLL格式语料
   - 将`ctb8tree.txt`原始语料文件转化为`CoNLL-X`格式。具体生成方式见 `MakeCollXFile.java]`
   - 格式参考：[http://ilk.uvt.nl/conll/#dataformat]
- embeddings文件
   - 使用DependencyParser训练必须指定的文件，具体生成方式见 `MakeEmbeddingsFile.java`]`，需要注意的是该文件的列数要比训练命令中`-embeddingSize`参数大1

训练需要的jar文件，放到`train/lib`下，其他的语料文件放到`/train`目录下，具体参考`TrainParser.java`，训练命令为
```shell
java -cp $CLASSPATH:lib/* edu.stanford.nlp.parser.nndep.DependencyParser \
    -trainFile trainFile \
    -embedFile wordEmbeddingFile \
    -embeddingSize 50 \
    -model modelOutputFile.txt.gz \
    ‑trainingThreads 20
```
### 参考
- [DependencyParser train 的官方文档](http://nlp.stanford.edu/nlp/javadoc/javanlp-3.5.0/edu/stanford/nlp/parser/nndep/DependencyParser.html)
- 