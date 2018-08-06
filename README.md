# Multi-view to Novel view: <br/>Synthesizing Novel Views with Self-Learned Confidence 

## Descriptions
This project is a TensorFlow implementation of [**Multi-view to Novel view: Synthesizing novel views with Self-Learned Confidence**](https://shaohua0116.github.io/Multiview2Novelview/), which is published in ECCV 2018. We provide codes, datasets, and checkpoints. 

In this work, we address the task of **multi-view novel view synthesis**, where we are interested in synthesizing a target image with an arbitrary camera pose from given source images. We propose an end-to-end trainable framework that learns to exploit multiple viewpoints to synthesize a novel view without any 3D supervision. Specifically, our model consists of a **flow prediction module** (*flow predictor)* and a **pixel generation module** (*recurrent pixel generator*) to directly leverage information presented in source views as well as hallucinate missing pixels from statistical priors. To merge the predictions produced by the two modules given multi-view source images, we introduce a **self-learned confidence aggregation mechanism**. An illustration of the proposed framework is as follows.

<p align="center">
    <img src="https://shaohua0116.github.io/Multiview2Novelview/img/model.jpg" height="320"/>
</p>

We evaluate our model on images rendered from 3D object models ([ShapeNet](https://www.shapenet.org/)) as well as real and synthesized scenes ([KITTI](http://www.cvlibs.net/datasets/kitti/) and [Synthia](http://synthia-dataset.net/)). We demonstrate that our model is able to achieve state-of-the-art results as well as progressively improve its predictions when more source images are available.

\*This code is still being developed and subject to change.


## Prerequisites

- Python 2.7
- [Tensorflow 1.3.0](https://github.com/tensorflow/tensorflow/tree/r1.0)
- [SciPy](http://www.scipy.org/install.html)
- [NumPy](http://www.numpy.org/)
- [colorlog](https://pypi.python.org/pypi/colorlog)
- [cv2](https://pypi.python.org/pypi/opencv-python)
- [h5py](http://docs.h5py.org/en/latest/build.html#install)
- [Pillow](https://pillow.readthedocs.io/en/latest/installation.html#basic-installation)

## Datasets
All datasets are stored as HDF5 files, and the links are as follows. Each data point (HDF5 group) contains an image and its camera pose.

### ShapeNet
<p align="center">
    <img src="./asset/shapenet_example.gif" width="800"/>
</p>
Download from the links ([car](https://drive.google.com/open?id=1vrZURHH5irKrxPFuw6e9mZ3wh2RqzFC9) and [chair](https://drive.google.com/open?id=1-IbmdJqi37JozGuDJ42IzOFG_ZNAksni)) and put the file to this directory `./datasets/shapenet`.

### KITTI
<p align="center">
    <img src="./asset/kitti_example.gif" width="800"/>
</p>
Download from this [link](https://drive.google.com/open?id=1LT3WoHxdCycu4jTxCGc1vGYpdRWridFH) and put the file to this directory `./datasets/kitti`.

### Synthia
<p align="center">
    <img src="./asset/synthia_example.gif" width="800"/>
</p>
Download from this [link](https://drive.google.com/open?id=1Fxv5r7oeG0PHgR42S5pHNvyl2pJN739H) and put the file to this directory `./datasets/synthia`.

## Usage
After downloading the datasets, we can start to train models with the following command:
### Train
```bash
$ python trainer.py  --batch_size 8 --dataset car --num_input 4
```
- Arguments
    - --prefix: a nickname for the training
    - --dataset: choose among `car`, `chair`, `kitti`, and 'synthia`. You can also add your own datasets.
    - Checkpoints: specify the path to a pre-trained checkpoint
        - --checkpoint: load all the parameters including the flow and pixel modules and the discriminator.
        - --checkpoint\_p: load only the parameters of the pixel module
        - --checkpoint\_f: load only the parameters of the flow module
        - --checkpoint\_g: load the parameters of both the flow and pixel module
        - --checkpoint\_d: load the parameters of the discriminator
    - Logging
        - --log\_setp: the frequency of outputing log info (`[train step  681] Loss: 0.51319 (1.896 sec/batch, 16.878 instances/sec)`)
        - --ckpt\_save\_step: the frequency of saving a checkpoint
        - --test\_sample\_step: the frequency of performing testing inference during training (default 100)
        - --write\_summary\_step: the frequency of writing TensorBoard summaries (default 100)
    - Hyperparameters
        - --num\_input: the number of source images
        - --batch\_size: the mini-batch size (default 8)
        - --local\_confidence\_weight: the weight of the confidence prediction objective (refer to the paper)
        - --learning\_rate\_p: the learning rate of the pixel module
        - --learning\_rate\_f: the learning rate of the flow module
        - --learning\_rate\_d: the learning rate of the discriminator
        - --max\_steps: the max training iterations
    - Architecture
        - --num\_res\_block\_pixel: the number of residual block in the bottleneck of the pixel module
        - --num\_res\_block\_flow: the number of residual block in the bottleneck of the flow module
        - --num\_dis\_conv\_layer: the number of convolutional layers of the discriminator
        - --num\_conv\_layer: the number of convolutional layers of the encoder of both the flow and pixel modules
        - --num\_convlstm\_block: the number of residual ConvLSTM block of the pixel module
        - --num\_convlstm\_scale: how many innermost layers of the pixel module have a residual ConvLSTM connection
        - --norm\_type: the type of normalization such as batch norm, instance norm
    - GAN
        - --gan\_type: the type of GAN losses such as LS-GAN, WGAN, etc
        - --gan\_start\_step: start to optimize the GAN loss when the model is stable
        - --update\_rate: update G more frequently than D
    - Multi-scale: this is not reporeted in the paper
        - --num\_scale: the number of multi-scale flow prediction (1 means without multi-scale prediction)
        - --moving\_weight: gradually learn each scale from coarse to fine

### Interpret TensorBoard
Launch Tensorboard and go to the specified port, you can see differernt losses in the **scalars** tab and plotted images in the **images** tab. The plotted images could be interpreted as follows.

<p align="center">
    <img src="./asset/TB.jpg" width="1024"/>
</p>

### Test
We can also evaluate trained models or the checkpoints provided by the authors with the following command:
```bash
$ python evaler.py --dataset car --data_id_list ./testing_tuple_lists/id_car_test.txt --loss [--train_dir /path/to/the/training/dir/ OR --checkpoint /path/to/the/trained/model] --write_summary --summary_file log_car.txt --plot_image --output_dir img_car
```
- Arguments
    - Id list
        - --data_id_list: specify a list of data point that you want to evaluate (ex. `./testing_tuple_lists/id_car_test.txt`)
    - Task
        - --loss: report the loss
        - --plot_image: rendered predicted images
    - Output
        - --quiet: only display the final report
        - --write_summary: write the summary of this evaluation as a text file
        - --summary_file: the path to the summary filw
        - --output_dir: the output dir of plotted images

## Result

### ShapeNet Cars
<p align="center">
    <img src="https://shaohua0116.github.io/Multiview2Novelview/img/results/car.jpg" width="640"/>
</p>
More results for ShapeNet cars (1k randomly samlped results from all 10k testing data): [link](https://shaohua0116.github.io/Multiview2Novelview/results/car_sampled/car_results_1k.html)

### ShapeNet Chairs
<p align="center">
    <img src="https://shaohua0116.github.io/Multiview2Novelview/img/results/chair.jpg" width="640"/>
</p>
More results for ShapeNet cars (1k randomly samlped results from all 10k testing data): [link](https://shaohua0116.github.io/Multiview2Novelview/results/car_sampled/car_results_1k.html)

### Scenes: KTIIT and Synthia
<p align="center">
    <img src="https://shaohua0116.github.io/Multiview2Novelview/img/results/scene.jpg" width="640"/>
</p>

## Checkpoints
We provide checkpoints and evaluation report files of our models for all experiments.
- ShapeNet Cars
- ShapeNet Chairs
- KITTI
- Synthia

## Related work
- [Multi-view 3D Models from Single Images with a Convolutional Network](https://arxiv.org/abs/1511.06702) in CVPR 2016
- [View Synthesis by Appearance Flow](https://arxiv.org/abs/1605.03557) in ECCV 2016
- [Transformation-Grounded Image Generation Network for Novel 3D View Synthesis](https://arxiv.org/abs/1703.02921) in CVPR 2017
- [Weakly-supervised Disentangling with Recurrent Transformations for 3D View Synthesis](https://arxiv.org/abs/1601.00706) in NIPS 2015

## Cite the paper
If you find this useful, please cite
```
@inproceedings{sun2016multiview,
  title={Multi-view to Novel view: Synthesizing Views via Self-Learned Confidence},
  author={Sun, Shao-Hua and Huh, Minyoung and Liao, Yuan-Hong and Zhang, Ning and Lim, Joseph J},
  booktitle={European conference on computer vision},
  year={2018},
}
```

## Authors
[Shao-Hua Sun](http://shaohua0116.github.io/), [Minyoung Huh](http://minyounghuh.com/), [Yuan-Hong Liao](https://andrewliao11.github.io/), [Ning Zhang](https://people.eecs.berkeley.edu/~nzhang/), and [Joseph J. Lim](http://www-bcf.usc.edu/~limjj/)
