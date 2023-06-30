

<h1 align="center"> iCatcher+ Pages</h1>

<!---
<a href="https://github.com/icatcherplus/icatcher_plus" class="btn btn-primary">View on GitHub</a>
-->
<!---
Repository found [here](https://github.com/icatcherplus/icatcher_plus).
-->
<!---
[![button](https://github.com/icatcherplus/icatcherplus.github.io/blob/main/images/github-mark.png?raw=true)](https://github.com/icatcherplus/icatcher_plus) [![button2](https://github.com/icatcherplus/icatcherplus.github.io/blob/main/images/quest_logo.png?raw=true)](https://quest.mit.edu/)
-->
<p float="left">
  <a href="https://github.com/icatcherplus/icatcher_plus">
    <img src="https://github.com/icatcherplus/icatcherplus.github.io/blob/main/images/github-mark.png?raw=true" width="116">
  </a>
  &nbsp; &nbsp; &nbsp; &nbsp;
  <a href="https://quest.mit.edu/">
    <img src="https://github.com/icatcherplus/icatcherplus.github.io/blob/main/images/quest_logo.png?raw=true" width="350">
  </a>
</p>

# Introduction

<!---feel free to change to whatever, this is all very loose... copied installation section from readme -->

[iCatcher+](https://doi.org/10.1177/25152459221147250) is a tool utilized for performing automatic annotation of 
discrete infant gaze directions from videos collected in the lab, field or online (remotely). This process is highly
customizable; users can choose between different face detectors, infant face classifiers, and gaze direction classifiers
to create an optimized pipeline that produces the most accurate infant gaze annotations.

<img src="https://github.com/icatcherplus/icatcherplus.github.io/blob/main/gaze.gif?raw=true" />

<!---
![](https://github.com/icatcherplus/icatcherplus.github.io/blob/main/gaze.gif)
-->
# Quick Installation (Windows, Linux, Mac)
This option will let you use iCatcher+ with minimum effort, but only for predictions (inference).
We strongly recommend using a virtual environment such as [Miniconda](https://conda.io) or [virtualenv](https://pypi.org/project/virtualenv/) before running the command below.

`pip install icatcher`

You will also need [ffmpeg](https://www.ffmpeg.org/) installed in your system and available (if you used conda, you can quickly install it with `conda install -c conda-forge ffmpeg`).

Note1:
If you require speedy performance, prior to installing icatcher you should install [PyTorch](https://pytorch.org/) with GPU support (see [here](https://pytorch.org/get-started/locally/) for instructions). This assumes you have a supported GPU on your machine.

Note2:
When using iCatcher+ for the first time, neural network model files will automatically be downloaded to a local cache folder. To control where they are downloaded to set the "ICATCHER_DATA_DIR" environment variable.

# Usability
In order to increase the accuracy of the iCatcher+ gaze coding system, several design decisions were made that, while 
increasing accuracy, may affect other aspects of iCatcher+ such as increasing the run time. As a user of iCatcher+, the 
desire for speed vs. accuracy, as well as the user’s resources (CPU vs. GPU), are all taken into account, and the user 
may customize the iCatcher+ pipeline in order to align with whatever result they prioritize.

Two tracks of the pipeline have been created based on the compute resources of the user.


## Track A (GPU)
Track A (the recommended track) relies on a GPU to run the iCatcher+ pipeline. 

INSERT DEFAULT PIPELINE INFO/ACCURACY/RUNTIME HERE

## Track B (CPU)
If a GPU is not available, the iCatcher+ pipeline defaults to a CPU version of the track, which increases the run time
of the annotation process. As a result of this increase in run time, several customizations have been added to try and 
keep Track B as a viable option. 

### Face Detection
The default face detection model used in the iCatcher+ pipeline is **RetinaFace**. This model is compute intensive and
results in long run times when used in the CPU track. For this reason, another face detection model (**opencv_dnn**) is
offered. This model is less accurate, but allows for quicker run times if that is the priority.

If you elect to prioritize accuracy and stick with the default RetinaFace face detector, there are certain routes you
can take to decrease the natural long run time.

### Parallel Processing
DISCUSS SAVING CPUS, BATCH INFERENCE

### Skipping Frames
DISCUSS CORRELATION BETWEEN SKIPPING FRAMES AND DECREASE RUN TIME, AT COST OF ACCURACY
 

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



