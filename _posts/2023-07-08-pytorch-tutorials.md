## Pytorch

I'd like to build a little language model of my own to see how well a really simple approach would work. I'm going to run as quickly as I can through some pytorch tutorials to get to the point where I can build a little neural net to start dicking with it. This post is my notes as I go through those.

The tutorial for neural nets is [here](https://pytorch.org/tutorials/beginner/blitz/neural_networks_tutorial.html), but I'm going to do the tutorials it suggests for PyTorch too.

### Tensors

A data class, essentially an n-dimensional array but optimised for running on GPUs. Compatible with NumPy arrays.

#### Creation
There are loads of factory functions on torch that take a tuple containging the dimensions you want: 
* `torch.ones((2, 3))`
* `torch.zeros((2, 3))`
* `torch.rand((2, 3))`

There are params on the functions to tweak stuff: 
`torch.ones((2, 3, dtype=int))`

You can also create tensors based on the shape & datatype of other tensors:
`torch.ones_like(another_tensor)`

#### From a CSV file uising pandas
```python
import pandas

# Gives you a Dataframe instance with the data
data = pandas.read_csv("filename")

# Row 0, field 1 - str if it's string
# numpy.int64 if all values of this field are int
# numpy.float64 if any of them are floats
print(data.iloc[0,1]) 

# If all values in the Dataframe are numerical
tensor = torch.tensor(data)
```

#### Attributes

```python
tensor = torch.rand(3, 4)
tensor.shape # ([3, 4])
tensor.dtype # torch.float32
tensor.device # cpu
```
#### Operations

Lots of operations exist; linear, arithmetic, transpose, slice, etc.

Operations on tensors can be run on GPU, but by default go on CPU.  To use the GPU:
```python
if torch.cuda.is_available():
    tensor = tensor.to("cuda")
```

```python
# Transpose
tensor.T 

# Element-wise product; new tensor
tensor1 * tensor2
tensor1.mul(tensor2)

# Functions that modify the tensor
tensor1.mul_(tensor2)

# Single element tensor to python value
tensor.item()
```

### Datasets, Dataloaders

The classes that pytorch uses for managing datasets are:
* `torch.utils.data.DataSet` - store & load samples & labels
* `torch.utils.data.DataLoader` - Wrap an iterable around `Dataset` for easy use

#### Prebuilt datasets 
A number of public datasets are made available in a module called `torchvision`:
```python
from torchvision import datasets
dir(datasets)
['CIFAR10', 'CIFAR100', 'CLEVRClassification', 'CREStereo', 'Caltech101', 'Caltech256', 'CarlaStereo', 'CelebA', 'Cityscapes', 'CocoCaptions', 'CocoDetection', 'Country211', 'DTD', 'DatasetFolder', 'EMNIST', 
'ETH3DStereo', 'EuroSAT', 'FER2013', 'FGVCAircraft', 'FakeData', 'FallingThingsStereo', 'FashionMNIST', 'Flickr30k', 'Flickr8k', 'Flowers102', 'FlyingChairs', 'FlyingThings3D', 'Food101',
 'GTSRB', 'HD1K', 'HMDB51', 'INaturalist', 'ImageFolder', 'ImageNet', 'InStereo2k', 'KMNIST', 'Kinetics', 'Kitti', 'Kitti2012Stereo', 'Kitti2015Stereo', 'KittiFlow', 'LFWPairs', 'LFWPeople', 'LSUN', 'LSUNClass', 'MNIST', 'Middlebury2014Stereo', 'MovingMNIST', 'Omniglot', 'OxfordIIITPet', 'PCAM', 
 'PhotoTour', 'Places365', 'QMNIST', 'RenderedSST2', 'SBDataset', 'SBU', 'SEMEION', 'STL10', 'SUN397', 'SVHN', 'SceneFlowStereo', 'Sintel', 'SintelStereo', 'StanfordCars', 'UCF101', 'USPS', 'VOCDetection', 'VOCSegmentation', 'VisionDataset', 'WIDERFace', '__all__', '__builtins__', '__cached__', '__doc__', 
 '__file__', '__getattr__', '__loader__', '__name__', '__package__', '__path__', '__spec__', '_optical_flow', '_stereo_matching', 'caltech', 'celeba', 'cifar', 'cityscapes', 'clevr', 'coco', 'country211', 'dtd', 'eurosat', 'fakedata', 'fer2013', 'fgvc_aircraft', 
 'flickr', 'flowers102', 'folder', 'food101', 'gtsrb', 'hmdb51', 'imagenet', 'inaturalist', 'kinetics', 'kitti', 'lfw', 'lsun', 'mnist', 'moving_mnist', 'omniglot', 'oxford_iiit_pet', 'pcam', 'phototour', 'places365', 'rendered_sst2', 'sbd', 'sbu', 'semeion', 'stanford_cars', 'stl10', 'sun397', 'svhn', 'ucf101', 'usps', 'utils', 'video_utils', 'vision', 'voc', 'widerface']
```

The datasets are classes that can be constructed to download and load the data:
```python
training_data = datasets.FashionMNIST(
    root="data",
    train=True,
    download=True,
    transform=ToTensor()
)
```

#### Your own dataset

Inherit from `Dataset`.  You need to provide implementations for:
* `__init__`: duh
* `__len__`: duh
* `__getitem__(self, idx)`: returns a tuple of `data, label`

eg. 
```python
class MyDataset(Dataset):
    '''
    Dataset that reads from a CSV file, where column 0 is the label and the 
    remainder of the fields are the data.
    '''
    def __init__(self, csv_file):
        self.data = pd.read_csv(csv_file)
    #    self.data.sort_values('one', inplace=True)
    
    def __len__(self):
        return len(self.data)
    
    def __getitem__(self, idx):
        '''
        Return a tuple of (features, label)
        '''
        assert isinstance(idx, int)
        row = self.data.iloc[idx]
        return row[1:].to_list(), row[0]
```

#### DataLoader

Datasets get wrapped in DataLoaders that provide batching, n-processor, shuffling (to avoid overfit), etc.

```python
dataloader = DataLoader(mydataset, batch_size=4, shuffle=True, num_workers=0)

for i_batch, sample_batched in enumerate(dataloader):
    print(i_batch, sample_batched)
```

### Transforms

Used for data cleanup before using the data for training. `Dataset` have a `transform` to transform the features, and `target_transform` for the labels. `torchvision.transforms`  has a lib of them.

