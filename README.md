# Tensorflow Image Classification
This repo demonstrates an image classification system for any set of images reusably. It uses tensorflow to create a neural network and do the image classification, and imagemagick to preprocess images. It then will generate data files for the images and labels for use withtn tensorflow. 

## Prerequisites
Make sure you have:
* Tensorflow
* Imagemagick
* Wand (python imagemagick wrapper)
* Set of pre-classified images (both a training set and a test set)
 
## How It Works

1. Put images into ```data/input/```, seperating them into different directories for training data and test data (```data/input/training``` and ```data/input/test``` for example)
2. Further seperate images into directories representing different classifications (```data/input/[training/test]/class1```, ```data/input/[training/test]/class2``` etc...)
3. Modify the ```trainingset.json``` and ```testset.json``` to set the height and width of images, as well as adding the classification directories made in (2), giving them a label. The json files have an example already written in, assuming 2 classifications in directories ```data/input/[training/test]/positive``` and ```data/input/[training/test]/negative``` for a binary classification (if output files are modified, their location will also need to be modified in ```tf_data.py```
4. Run ```./build_data_set [dataset].json``` which will build the imageset and labelset into the outputfiles specified in the json file (```output/TF[test/train].[labels & images].gz``` by default)
5. Run ```python classify.py``` to train the model and test the data set. 
