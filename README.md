# mmocr-1.0.0

[mmocr-1.0.0](https://github.com/open-mmlab/mmocr/tree/v1.0.0).

# 接口(app.py)
/mnt/sdb/jushi/ai/mmocr/mmocr-1.0.0/OcrDemo/app.py
## /train/start
1. 数据格式转换：mmocr有一套独立的数据组织格式，因此需要将网页上标注的数据格式转换为mmocr格式，该函数已写进train/start中，只要数据集在路径下，启动训练后即进行数据集格式转换。数据集转换函数在trainNet.py中，为coco2mmocr100,此外，还需要为检测模型和识别模型准备配置文件，定义在trainNet.py的prepareConfig函数中。trainNet.py路径/mnt/sdb/jushi/ai/mmocr/mmocr-1.0.0/tools/trainNet.py

2. 先训练检测模型，训练50个epoch,batch size为4，修改epoch，batch size在/mnt/sdb/jushi/ai/mmocr/mmocr-1.0.0/configs/textdet/dbnet/db_res50_XY.py。如果修改batch_size需要在loops.py(/home/jushi/anaconda3/envs/testV04/lib/python3.7/site-packages/mmengine/runner/loops.py)第146行将batch_size=4改掉。

3. 后训练识别模型，训练20个epoch，观察到识别模型在信云ocr数据集上一般10个epoch左右loss收敛为0，batch size为128，修改epoch,batch size的路径为/mnt/sdb/jushi/ai/mmocr/mmocr-1.0.0/configs/textrecog/satrn/satrn_shallow-small_XY.py。如果修改batch_size将loops.py第159行batch_size=128改掉。

4. 为了避免在服务器上保存太多的checkpoints，每训练5个epoch会对模型进行验证并保存当前的.pth，要求将总迭代次数设为5的倍数。验证的指标保存在status.json中，当前epoch(det_epoch,rec_epoch)，损失函数(det_loss,rec_loss)以及最大迭代次数(det_totalEpoch,rec_totalEpoch)，当前处理图片数(detimageProgress,recimageProgress)以及总图片数(TotalTrainDetImageList,TotalTrainRecImageList)等都会保存到status.json中。每一次重新训练，status.json都会重写。这样做的好处是方便status函数访问状态时返回当前进度和状态。

5. 训练完成后，模型以及配置文件会保存在model/pthModel.zip中。

6. 检测模型验证指标：precision, recall, hmean($hmean=\frac{1}{P}+\frac{1}{R}$)

7. 识别模型验证指标：word_acc, word_acc_ignore_case, word_acc_ignore_symbol, char_recall, char_precision. 具体解释见[mmocr](https://github.com/open-mmlab/mmocr/blob/main/docs/zh_cn/basic_concepts/evaluation.md).

8. 和轩哥交流，在前端展示三个指标，检测模型两个：precision和recall，识别模型一个：char_precision。但在status接口以及status report中，上述8个指标全部传回，方便以后在前端改进。

9. 当前epoch, loss, 以及检测识别模型的几个指标都在训练或验证过程中直接写入status.json中，在mmengine训练和验证过程中直接写入json，写入源码位置/home/jushi/anaconda3/envs/testV04/lib/python3.7/site-packages/mmengine/runner/loops.py。

10. runTest==1时，会在训练完成后立即测试，此时需要测试集图片在训练模型路径下，会启动测试。与/test/start/接口处理方式相同，会在路径下写入status_test.json，记录当前测试图片进度，总图片数以及状态。

## /train/stop
停止训练模型，根据projectID，kill为该项目训练分配的pid

## /test/start
1. 将test过程中的状态保存在status_test.json下，记录测试图片进度(testImage)，总图片数(totalTestImage)以及状态。

2. 加载checkpoints时读取train过程的status.json，从该json中得到最大迭代次数，然后直接加载最大迭代次数的模型，因此必须训练成功后再测试，不然加载不了模型。

3. 采用one by one的方式测试，即一张图片先检测后识别，结果保存在testResult.json中。

## /test/stop
停止测试模型，根据projectID，kill为该项目测试分配的pid

## /status
1. 返回状态，在训练过程中，status.json中保存的状态如下：
   ```
   "status": "成功", #当前状态，有：成功，失败，训练检测模型中，检测模型训练完毕，验证检测模型中，检测模型验证完毕，训练识别模型中，识别模型训练完毕，验证识别模型中，识别模型识别完毕
    "det_totalEpoch": 50, #检测模型的总迭代次数
    "TotalTrainDetImageList": 274, #需要检测的总图片数
    "TotalTrainRecImageList": 4932, #需要识别的总图片数
    "detimageProgress": 274, #当前训练的检测图片进度
    "det_epoch": 50, #当前训练epoch
    "det_loss": 0.5657,#当前检测模型损失
    "precision": 0.9992,#精度
    "recall": 0.9996,#召回率
    "hmean": 0.9994,#1/P+1/R
    "rec_totalEpoch": 20,#识别模型的总迭代次数
    "recimageProgress": 4932,#当前训练的识别图片进度
    "rec_epoch": 20,#当前训练epoch
    "rec_loss": 0.0,#识别损失
    "word_acc": 1.0,
    "word_acc_ignore_case": 1.0,
    "word_acc_ignore_case_symbol": 1.0,
    "char_recall": 1.0,
    "char_precision": 1.0
   ```
2. status接口的参数基本都是从status.json或status_test.json中得到

## /train/checkFile
1. type=model:检查模型是否训练完成并保存在model/pthModel.zip中。返回pthModel.zip的大小

2. type=testResult: 检查是否测试完成并保存为testResult.json。返回testResult.json的大小

## /test/checkFile
检查测试是否完成。返回testResult.json的大小

## /train/download
1. downloadType=model 下载训练完的模型

2. downloadType=testResult，下载测试后的结果testResult.json

## /test/download
下载测试后的结果testResult.json

## getEpoch
1. 该函数的作用是从模型训练的log文件中读取当前的epoch, loss, 后来直接从mmengine的源码中读取，这个函数就没用了。

2. 改变了mmengine为log文件命名的方式，统一命名为log/record.log。原来是以当前时间作为文件名。修改源码位置：/home/jushi/anaconda3/envs/testV04/lib/python3.7/site-packages/mmengine/runner/runner.py
   
