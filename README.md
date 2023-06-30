
<p align="center">
  <a href="https://quest.mit.edu/">
    <img src="https://github.com/icatcherplus/icatcherplus.github.io/blob/main/images/quest_logo_cropped.png?raw=true" height="80">
  </a>
  &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;
  <a href="https://github.com/icatcherplus/icatcher_plus">
    <img src="https://github.com/icatcherplus/icatcherplus.github.io/blob/main/images/github-mark.png?raw=true" height="80">
  </a>
</p>

---
# About iCatcher+

<!---feel free to change to whatever, this is all very loose... copied installation section from readme -->

[iCatcher+](https://doi.org/10.1177/25152459221147250) is a tool utilized for performing automatic annotation of 
discrete infant gaze directions from videos collected in the lab, field or online (remotely). This process is highly
customizable; you can choose between different face detectors, infant face classifiers, and gaze direction classifiers
to create an optimized pipeline that produces the most accurate infant gaze annotations.

<img src="https://github.com/icatcherplus/icatcherplus.github.io/blob/main/gaze.gif?raw=true" align="center"/>

<!---
![](https://github.com/icatcherplus/icatcherplus.github.io/blob/main/gaze.gif)
-->

## Quick Installation (Windows, Linux, Mac)
This option will let you use iCatcher+ with minimum effort, but only for predictions (inference).
We strongly recommend using a virtual environment such as [Miniconda](https://conda.io) or [virtualenv](https://pypi.org/project/virtualenv/) before running the command below.

`pip install icatcher`

You will also need [ffmpeg](https://www.ffmpeg.org/) installed in your system and available (if you used conda, you can quickly install it with `conda install -c conda-forge ffmpeg`).

Note1:
If you require speedy performance, prior to installing icatcher you should install [PyTorch](https://pytorch.org/) with GPU support (see [here](https://pytorch.org/get-started/locally/) for instructions). This assumes you have a supported GPU on your machine.

Note2:
When using iCatcher+ for the first time, neural network model files will automatically be downloaded to a local cache folder. To control where they are downloaded to set the "ICATCHER_DATA_DIR" environment variable.
---
# iCatcher+ Pipeline
The iCatcher+ pipeline can be segmented into three distinct components: face detection, face classification, and gaze
classification. Several tests have been run on each of these components to optimize overall performance on annotation
accuracy. Each of these components has plug-and-play options to allow for higher accuracy and quicker results in varying
circumstances. Below, the default iCatcher+ pipeline is defined:

### Face Detector
iCatcher+ utilizes an out-of-box face detector based on [RetinaFace](https://arxiv.org/abs/1905.00641), a robust state 
of the art pretrained model with a resnet50 backbone. A variation of RetinaFace that allows for batch inference is
used in order to decrease compute time, and can be found [here](https://github.com/elliottzheng/batch-face).

### Face Classifier
Although a face classifier has been trained to distinguish between infant faces and adult faces within input videos
(and is readily available for use with the flag -`-use_fc_model`), the best approach tends to be with the default "lowest 
face" selector. Since studies typically require infants' faces to be below parents, this method selects the face based
on a mix of its bounding box's y-coordinate and the height:width ratio being close to 1 (parents are normally instructed to look away 
from the camera during studies, so their relative bounding boxes tend to be more rectangular).

INSERT DEMONSTRATION SIDE BY SIDE IMAGE OF RATIO BEING USED HERE

### Gaze Classifier
KHALED INSERT DETAILS HERE

---
# Usability
After installing iCatcher+, it can be easily run by following [these instructions](https://github.com/icatcherplus/icatcher_plus#running-icatcher).
**ADD MORE OR LESS HERE DEPENDING ON DEIRDRE UI**

In order to increase the accuracy of the iCatcher+ gaze coding system, several design decisions were made that, while 
increasing accuracy, may affect other aspects of iCatcher+ such as increasing the run time. As a user of iCatcher+, the 
desire for speed vs. accuracy, as well as the user’s resources (CPU vs. GPU), are all taken into account, and you 
may customize the iCatcher+ pipeline in order to align with whatever result you prioritize.

Two tracks of the pipeline have been created based on the compute resources of the user: a GPU track and a CPU track.

---
## Track A (GPU)
Track A (the recommended track) relies on a GPU to run the iCatcher+ pipeline. This track utilizes the default pipeline
components specified above, but you must insert the flag `--gpu_id #`, with the # corresponding to the GPU you
would like to use (insert 0 if your system has a single GPU).

When tested on 96 videos within the [Lookit dataset](https://osf.io/ujteb/), each averaging a runtime of 9 minutes 20 seconds, Track A automatically annotated each video 
in about **20 minutes**. The iCatcher+ default pipeline was able to correctly annotate approximately 91% of these frames
when compared to two human coders. 

---
## Track B (CPU)
If a GPU is not available, the iCatcher+ pipeline defaults to a CPU version of the track, which increases the run time
of the annotation process. As a result of this increase in run time, several customizations have been added to try and 
keep Track B as a viable option. 

### Face Detection
The default face detection model used in the iCatcher+ pipeline is **RetinaFace**. This model is computationally
intensive and results in long run times when used in the CPU track. For this reason, another face detection model 
(**an OpenCV DNN**) is offered. This model is less accurate, but allows for quicker run times if that's your priority.

If you elect to prioritize accuracy and stick with the default RetinaFace face detector, there are certain routes you
can take to decrease the associated long run time.

#### Parallel Processing
Parallel processing is used by default when RetinaFace is run on CPU. This will split the frames into batches that will
be processed on each core of your system. After testing, a batch size of 16 frames was chosen as the default setting. 
This can be customized using the `--fd_batch_size` flag. If you don't want all of your cores to be utilized in the 
parallelization process (for instance if you have other things to run or work on), you can specify an amount of CPUs 
to pull out of the pipeline through the `--num_cpus_saved` flag. One caveat of parallel processing input frames is that
the full video is buffered in order to divide it into batches and send out to each core. This process can be switched
off by changing the `--dont_buffer` flag, which may help with memory issues of loading the full video, but will increase
overall run time.

### Skipping Frames
When using RetinaFace, you can choose to alter the amount of frames you are running through the face detector by 
selecting an amount of frames to 'skip'. For the amount of frames you specify, the last known bounding box coordinates 
of an infant's face will be reused and fed back into the gaze classifier. Since videos are typically 30 FPS, infant 
faces tend to stay relatively stationary between frames. Skipping frames will decrease the run time of the
pipeline, but may variably decrease the accuracy.

---
## Flags
In order to see all of the available options iCatcher+ offers, you can run:

`icatcher --help`

Although this command will list all of the possible flags, there are certain flags that are important to know when 
working in Track A (GPU) vs. Track B (CPU).

| **Flags**                 | **Track(s)** | **Choices**                                            | **Default**                                | **Description**                                                                                                                                                    |
|---------------------------|--------------|--------------------------------------------------------|--------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| --gpu_id                  | A, B         | -1 (for CPU), 0 (or other number corresponding to GPU) | -1                                         | The GPU ID to use. The default is CPU, so this must be changed when using GPU.                                                                                     |
| --fd_model                | A, B         | retinaface, opencv_dnn                                 | retinaface                                 | The face detector model used within the iCatcher+ pipeline. opencv_dnn may be more suitable for cpu usage if speed is a greater priority than accuracy.            |
| --fd_confidence_threshold | A, B         | float                                                  | 0.9 (if retinaface) or 0.7 (if opencv_dnn) | The score confidence threshold that needs to be met for a bounding box to be accepted as a face. A higher score represents more stringent face requirements.       |
| --fd_batch_size           | A, B         | int                                                    | 16                                         | Corresponds to the number of frames fed into the RetinaFace face detector at one time for batch inference.                                                         |
| --num_cpus_saved          | B            | int                                                    | 0                                          | Specifies the number of CPUs you’d like to keep from being used in the RetinaFace face detection parallel processing.                                              |
| --fd_skip_frames          | B            | int                                                    | 0                                          | The number of frames to skip between each face detection. If frames are skipped, the last known bounding box is reused for the skipped frames.                     |
| --dont_buffer             | B            | True, False                                            | False                                      | When changed, frames will not be buffered, decreasing memory usage, but increasing processing time. Turning off the buffer also allows for live stream of results. |

---

