# Face Recognition using Tensorflow [![Build Status][travis-image]][travis]

[travis-image]: http://travis-ci.org/davidsandberg/facenet.svg?branch=master
[travis]: http://travis-ci.org/davidsandberg/facenet

This is a TensorFlow implementation of the face recognizer described in the paper
["FaceNet: A Unified Embedding for Face Recognition and Clustering"](http://arxiv.org/abs/1503.03832). The project also uses ideas from the paper ["A Discriminative Feature Learning Approach for Deep Face Recognition"](http://ydwen.github.io/papers/WenECCV16.pdf) as well as the paper ["Deep Face Recognition"](http://www.robots.ox.ac.uk/~vgg/publications/2015/Parkhi15/parkhi15.pdf) from the [Visual Geometry Group](http://www.robots.ox.ac.uk/~vgg/) at Oxford.

## 基于facenet的人脸识别DEMO
### 人脸识别face_recognition.py 的功能
如果模型已经存在（models/classifier.pkl），程序将自动进入实时人脸识别模式，现在设置的阈值是0.90的精确度，否则将被分类为未知。如果是mac os系统还会使用语音说出类别名。
* 按c键进入训练模式，跟着提示输入用户姓名，采集10秒的样本，开始上下左右转动头，什么也不输入将结束样本采集，接着执行训练，训练结束生成模型。
* 按空格键将进入暂停模式。
* 按q键将退出程序。
### 目录结构
```
facenet
├── images
│   ├── train           采集用于训练的样本
│   ├── train_mtcnn     裁剪人脸
│   ├── capture         实时捕捉
│   ├── capture_mtcnn   裁剪人脸
│   └── classifier      分类识别
├── models
│   ├── 20170511-185253 CASIA-WebFace
│   ├── 20170512-110547 MS-Celeb-1M
│   └── classifier.pkl  分类模型
├── src
│   ├── face_recognition.py
...
...
```
### 运行
``` bash
python face_recognition.py
```

## 使用
### 人脸检测
``` bash
python align/align_dataset_mtcnn.py images_align_mtcnn face_images --detect_multiple_faces True
```
### 人脸比较
``` bash
python compare.py ../models/20170512-110547 images_compare/ap_mtcnn/1.png images_compare/ap_mtcnn/2.png images_compare/ap_mtcnn/3.png images_compare/ap_mtcnn/4.png
```
### 人脸训练和分类（在自己的数据上）
``` bash
python classifier.py TRAIN images_classifier_mtcnn ../models/20170512-110547 ../models/classifier.pkl
python classifier.py CLASSIFY images_classifier_mtcnn ../models/20170512-110547 ../models/classifier.pkl
```

## Compatibility
The code is tested using Tensorflow r1.2 under Ubuntu 14.04 with Python 2.7 and Python 3.5. The test cases can be found [here](https://github.com/davidsandberg/facenet/tree/master/test) and the results can be found [here](http://travis-ci.org/davidsandberg/facenet).

## News
| Date     | Update |
|----------|--------|
| 2017-05-13 | Removed a bunch of older non-slim models. Moved the last bottleneck layer into the respective models. Corrected normalization of Center Loss. |
| 2017-05-06 | Added code to [train a classifier on your own images](https://github.com/davidsandberg/facenet/wiki/Train-a-classifier-on-own-images). Renamed facenet_train.py to train_tripletloss.py and facenet_train_classifier.py to train_softmax.py. |
| 2017-03-02 | Added pretrained models that generate 128-dimensional embeddings.|
| 2017-02-22 | Updated to Tensorflow r1.0. Added Continuous Integration using Travis-CI.|
| 2017-02-03 | Added models where only trainable variables has been stored in the checkpoint. These are therefore significantly smaller. |
| 2017-01-27 | Added a model trained on a subset of the MS-Celeb-1M dataset. The LFW accuracy of this model is around 0.994. |
| 2017&#8209;01&#8209;02 | Updated to code to run with Tensorflow r0.12. Not sure if it runs with older versions of Tensorflow though.   |

## Pre-trained models
| Model name      | LFW accuracy | Training dataset | Architecture |
|-----------------|--------------|------------------|-------------|
| [20170511-185253](https://drive.google.com/file/d/0B5MzpY9kBtDVOTVnU3NIaUdySFE) | 0.987        | CASIA-WebFace    | [Inception ResNet v1](https://github.com/davidsandberg/facenet/blob/master/src/models/inception_resnet_v1.py) |
| [20170512-110547](https://drive.google.com/file/d/0B5MzpY9kBtDVZ2RpVDYwWmxoSUk) | 0.992        | MS-Celeb-1M      | [Inception ResNet v1](https://github.com/davidsandberg/facenet/blob/master/src/models/inception_resnet_v1.py) |

## Inspiration
The code is heavily inspired by the [OpenFace](https://github.com/cmusatyalab/openface) implementation.

## Training data
The [CASIA-WebFace](http://www.cbsr.ia.ac.cn/english/CASIA-WebFace-Database.html) dataset has been used for training. This training set consists of total of 453 453 images over 10 575 identities after face detection. Some performance improvement has been seen if the dataset has been filtered before training. Some more information about how this was done will come later.
The best performing model has been trained on a subset of the [MS-Celeb-1M](https://www.microsoft.com/en-us/research/project/ms-celeb-1m-challenge-recognizing-one-million-celebrities-real-world/) dataset. This dataset is significantly larger but also contains significantly more label noise, and therefore it is crucial to apply dataset filtering on this dataset.

## Pre-processing

### Face alignment using MTCNN
One problem with the above approach seems to be that the Dlib face detector misses some of the hard examples (partial occlusion, silhouettes, etc). This makes the training set to "easy" which causes the model to perform worse on other benchmarks.
To solve this, other face landmark detectors has been tested. One face landmark detector that has proven to work very well in this setting is the
[Multi-task CNN](https://kpzhang93.github.io/MTCNN_face_detection_alignment/index.html). A Matlab/Caffe implementation can be found [here](https://github.com/kpzhang93/MTCNN_face_detection_alignment) and this has been used for face alignment with very good results. A Python/Tensorflow implementation of MTCNN can be found [here](https://github.com/davidsandberg/facenet/tree/master/src/align). This implementation does not give identical results to the Matlab/Caffe implementation but the performance is very similar.

## Running training
Currently, the best results are achieved by training the model as a classifier with the addition of [Center loss](http://ydwen.github.io/papers/WenECCV16.pdf). Details on how to train a model as a classifier can be found on the page [Classifier training of Inception-ResNet-v1](https://github.com/davidsandberg/facenet/wiki/Classifier-training-of-inception-resnet-v1).

## Pre-trained model
### Inception-ResNet-v1 model
A couple of pretrained models are provided. They are trained using softmax loss with the Inception-Resnet-v1 model. The datasets has been aligned using [MTCNN](https://github.com/davidsandberg/facenet/tree/master/src/align).

## Performance
The accuracy on LFW for the model [20170512-110547](https://drive.google.com/file/d/0B5MzpY9kBtDVZ2RpVDYwWmxoSUk) is 0.992+-0.003. A description of how to run the test can be found on the page [Validate on LFW](https://github.com/davidsandberg/facenet/wiki/Validate-on-lfw).
