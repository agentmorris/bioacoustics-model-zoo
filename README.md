# bioacoustics-model-zoo
Pre-trained models for bioacoustic classification tasks

# Basic usage

### List: 

List available models in the GitHub repo [bioacoustics-model-zoo](https://github.com/kitzeslab/bioacoustics-model-zoo/)
```
import torch
torch.hub.list('kitzeslab/bioacoustics-model-zoo')
```

### Load: 

Get a ready-to-use model object: choose from the models listed in the previous command
```
model = torch.hub.load('kitzeslab/bioacoustics-model-zoo','rana_sierrae_cnn')
```

### Inference:

`model` is an OpenSoundscape CNN object (or other class) which you can use as normal. 

For instance, use the model to generate predictions on an audio file: 

```
audio_file_path = './hydrophone_10s.wav'
scores = model.predict([audio_file_path],activation_layer='softmax')
scores
```

# Model list

### [Perch](https://tfhub.dev/google/bird-vocalization-classifier/4): 

Embedding and bird classification model trained on Xeno Canto

Example:

```python
import torch
model = torch.hub.load('kitzeslab/bioacoustics-model-zoo', 'Perch')
predictions = model.predict(['test.wav']) # predict on the model's classes
embeddings = model.generate_embeddings(['test.wav']) # generate embeddings on each 5 sec of audio
```

### [BirdNET](https://github.com/kahst/BirdNET-Analyzer)

Classification and embedding model trained on a large set of annotated bird vocalizations

Example: 

```python
import torch
m = torch.hub.load('kitzeslab/bioacoustics-model-zoo', 'BirdNET')
m.predict(['test.wav']) # returns dataframe of per-class scores
m.generate_embeddings(['test.wav']) # returns dataframe of embeddings
```

### [MixIT Bird SeparationModel](https://github.com/google-research/sound-separation/blob/master/models/bird_mixit/README.md)

Separate audio into channels potentially representing separate sources.

This particular model was trained on bird vocalization data. 

Example:

First, download the checkpoint and metagraph from the MixIt Github 
[repo](https://github.com/google-research/sound-separation/blob/master/models/bird_mixit/README.md):
install gsutil then run the following command in your terminal:

`gsutil -m cp -r gs://gresearch/sound_separation/bird_mixit_model_checkpoints .`

Then, use the model in python:
```python
import torch
# provide the local path to the checkpoint when creating the object
model = torch.hub.load(
    'kitzeslab/bioacoustics-model-zoo',
    'SeparationModel',
    checkpoint='/path/to/bird_mixit_model_checkpoints/output_sources4/model.ckpt-3223090'
) # creates 4 channels; use output_sources8 to separate into 8 channels

# separate opensoundscape Audio object into 4 channels:
# note that it seems to work best on 5 second segments
a = Audio.from_file('audio.mp3',sample_rate=22050).trim(0,5)
separated = model.separate_audio(a)

# save audio files for each separated channel:
# saves audio files with extensions like _stem0.wav, _stem1.wav, etc
model.load_separate_write('./temp.wav')
```

### [YAMNet](https://tfhub.dev/google/yamnet/1): 

Embedding model trained on AudioSet YouTube

Example:

```python
import torch
m = torch.hub.load('kitzeslab/bioacoustics-model-zoo', 'YAMNet')
m.predict(['test.wav']) # returns dataframe of per-class scores
m.generate_embeddings(['test.wav']) # returns dataframe of embeddings
```


### rana_sierrae_cnn: 

Detect underwater vocalizations of _Rana sierrae_, the Sierra Nevada Yellow-legged Frog

example: 
```python
import torch
m = torch.hub.load('kitzeslab/bioacoustics-model-zoo', 'rana_sierrae_cnn')
m.predict(['test.wav']) # returns dataframe of per-class scores
```

## Other automated detection tools for bioacoustics

### RIBBIT 

Detect sounds with periodic pulsing patterns. 

Implemented in [OpenSoundscape](https://opensoundscape.org) as
`opensoundscape.ribbit.ribbit()`.

- [Python notebooks demonstrating use](https://github.com/kitzeslab/ribbit_manuscript_notebooks)
- [Implementation for R](https://github.com/kitzeslab/r-ribbit)
- [Manuscript: Lapp et al 2021](https://conbio.onlinelibrary.wiley.com/doi/full/10.1111/cobi.13718)

### Accelerating and decelerating sequences

Detect pulse trains that accelerate, such as the drumming of Ruffed Grouse (_Bonasa umbellus_)

Implemented in [OpenSoundscape](https://opensoundscape.org) as

`opensoundscape.signal_processing.detect_peak_sequence_cwt()`. 

(note that in earlier versions of 
OpenSoundscape the module is named `signal` rather than `signal_processing`)

- [Python notebooks demonstrating use](https://github.com/kitzeslab/ruffed_grouse_manuscript_2022)
- [Manuscript: Lapp et al 2022](https://wildlife.onlinelibrary.wiley.com/doi/full/10.1002/wsb.1395)


## Troubleshooting 

### TensorFlow Installation in Python Environment

Installing TensorFlow can be tricky, and it may not be possible to have cuda-enabled tensorflow in the same environment as cuda-enabled pytorch. In this case, you can install a cpu-only version of tensorflow (`pip install tensorflow-cpu`). You may want to start with a fresh environment, or uninstall tensorflow and nvidia-cudnn-cu11 then reinstall pytorch with the appropriate nvidia-cudnn-cu11, to avoid having the wrong cudnn for PyTorch. 

Alternatively, if you want to use the TensorFlow Hub models with GPU acceleration, create an environment where you uninstall `pytorch` and `nvidia-cudnn-cu11` and install a cpu-only version (see [this page](https://pytorch.org/get-started/locally/) for the correct installation command). Then, you can `pip install tensorflow-hub` and let it choose the correct nvidia-cudnn so that it can use CUDA and leverage GPU acceleration. 

Installing tensorflow: Carefully follow the [directions](https://www.tensorflow.org/install/pip) for your system. Note that models provided in this repo might require the specific nvidia-cudnn-cu11 version 8.6.0, which could conflict with the version required for pytorch. 

### Error while Downloading TF Hub Models

Some of the models provided in this repo are hosted on the Tensorflow model hub. 

If you encounter the following error (or similar) when downloading a TensorFlow Hub model:

```
ValueError: Trying to load a model of incompatible/unknown type. '/var/folders/d8/265wdp1n0bn_r85dh3pp95fh0000gq/T/tfhub_modules/9616fd04ec2360621642ef9455b84f4b668e219e' contains neither 'saved_model.pb' nor 'saved_model.pbtxt'.
```

You need to delete the folder listed in the error message (something like `/var/folders/...tfhub_modules/....`). After deleting that folder, downloading the model should work. 

The issue occurs because TensorFlow Hub is looking for a cached 
model in a temporary folder where it was once stored but no longer exists. See relevant GitHub issue here: 
https://github.com/tensorflow/hub/issues/896
