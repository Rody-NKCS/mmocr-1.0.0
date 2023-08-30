# mmocr-1.0.0

## /train/start
1. 数据格式转换：mmocr有一套独立的数据组织格式，因此需要将网页上标注的数据格式转换为mmocr格式，该函数已写进train/start中，只要数据集在路径下，启动训练后即进行数据集格式转换。数据集转换函数在trainNet.py中，为coco2mmocr100,此外，还需要为检测模型和识别模型准备配置文件，定义在trainNet.py的prepareConfig函数中。

2. 先训练检测模型，训练50个epoch,batch size为4，修改epoch，batch size在/mnt/sdb/jushi/ai/mmocr/mmocr-1.0.0/configs/textdet/dbnet/db_res50_XY.py

3. 后训练识别模型，训练20个epoch，观察到识别模型在信云ocr数据集上一般10个epoch左右loss收敛为0，batch size为128，修改epoch,batch size的路径为/mnt/sdb/jushi/ai/mmocr/mmocr-1.0.0/configs/textrecog/satrn/satrn_shallow-small_XY.py

4. 每训练5个epoch会对模型进行验证，验证的指标保存在status.json中，训练的当前epoch，损失函数以及最大迭代次数，当前处理图片数等都会保存到status.json中。每一次重新训练，status.json都会重写。这样做的好处是方便status函数访问状态时返回当前进度和状态。

5. 训练完成后，模型以及配置文件会保存在model/pthModel.zip中。
