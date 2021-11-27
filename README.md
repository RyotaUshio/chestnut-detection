# chestnut-detection

This is an implementation of the method proposed in
> Keyhan Najafian, Alireza Ghanbari, Ian Stavness, Lingling Jin, Gholam Hassan Shirdel, and Farhad Maleki. Semi-self-supervised Learning Approach for Wheat Head Detection using Extremely Small Number of Labeled Samples. In *Proceedings of the IEEE/CVF International Conference on Computer Vision*, pages 1342-1351, 2021.

## Requirements
Python>=3.6.0  
NumPy>=??  
PyTorch>=1.7  
OpenCV>=??  
Albumentation>=1.1.0  

## Preparing datasets

1. Prepare your videos. Put video clips of backgrounds in `back` directory, fields in 'field'.

2. Choose 1 or more "representative frames" from each video clip of fields. Put them in `rep` directory.

3. Make pixel-wise labels for each representative image, just like you would do in semantic segmentation.

4. Export the labels in Pascal VOC format and put them in `mask` directory. If you have made the labels with [labelme](https://github.com/wkentaro/labelme), you can export them with the following command:
```
python labelme2voc.py [input_dir] dataset_voc --labels labels.txt
```
where`input_dir` is where you've put the `.json` files generated by labelme.
Now you've got `.npy` files in `dataset_voc/SegmentationClass`.
Note that `labelme2voc.py` is taken from [the original labelme repo](https://github.com/wkentaro/labelme/blob/main/examples/semantic_segmentation/labelme2voc.py) and fixed a little bit to handle wider range of images.

5. Create a text file named `labels.txt` and list class names in it. For example, it will look like this if you are working on chestnut detection:
```
nut(fine)
nut(empty)
burr
burr+nut
```

6. make sure your project directory is organized as below:

```
project_root
│   
└───back
│   │   background_video1.mp4
│   │   background_video2.mp4
│   │   ...
│   
└───field
│   │   field_video1.mp4
│   │   field_video2.mp4
│   │	...
│
└───rep # representative frames taken from each field video
│   │   field_video1_0031.png # [video_name]_[timestamp].png
│   │   field_video1_0118.png
│   │	field_video2_0005.png
│   │	...
│
└───mask
│   │   field_video1_0031.npy
│   │   field_video1_1018.npy
│   │	field_video2_0005.npy
│   │	...
│
└───labels.txt
```

5. Run the following to generate datasets.
```
python make_dataset.py -n [n_sample] -p [prob_1 prob_2 ... prob_n_class] --root path/to/project_root -o composite --verbose --bbox --cuda  --domain-adaptation domain_adaptation_1
```
You will see two datasets `composite` and `domain_adaptation_1` have been made.

6. Make `.yaml` file for YOLOv5 training with the dataset you've made.
```
# composite.yaml
path: path/to/project_root/composite/
train: images/all
val: images/all
test:

nc: 4
names: ['nut(fine)', 'nut(empty)', 'burr', 'burr+nut']
```

```
# domain_adaptation_1.yaml
path: path/to/project_root/domain_adaptation_1/
train: images/all
val: images/all
test:

nc: 4
names: ['nut(fine)', 'nut(empty)', 'burr', 'burr+nut']
```

7. Train YOLOv5.
```
git clone https://github.com/ultralytics/yolov5
cd yolov5
pip install -r requirements.txt
python train.py --img 640 --batch 16 --epochs 3 --data composite.yaml --weights yolov5s.pt
python train.py --img 640 --batch 16 --epochs 3 --data domain_adaptation_1.yaml --weights []
```

8. Make pseudo labels with the last model you've trained.

9. Train the model further with the pseudo labels.

