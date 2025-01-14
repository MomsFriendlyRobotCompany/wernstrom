![KITTI](https://raw.githubusercontent.com/MomsFriendlyRobotCompany/wernstrom/master/pics/wernstrom.png)

![GitHub](https://img.shields.io/github/license/MomsFriendlyRobotCompany/wernstrom)
![PyPI - Python Version](https://img.shields.io/pypi/pyversions/wernstrom)
![PyPI](https://img.shields.io/pypi/v/wernstrom)
![PyPI - Downloads](https://img.shields.io/pypi/dm/wernstrom?color=aqua)

# wernstrom

> So this package is a modification for what I use and a specific set of KITTI
> data that doesn't appear to follow the standard KITTI format.
> *Use at own risk*, your computer might melt or become sentient and
> take over the world!

This package provides a minimal set of tools for working with the KITTI dataset [[1]](#references) in Python. So far only the raw datasets and odometry benchmark datasets are supported.

## Layout


```
- Kitti dataset
  +- calib
  | +- 0000.txt
  | +- 00NN.txt
  +- oxts
  | +- 0000.txt
  | +- 00NN.txt
  +- image_00 (not used)
  +- image_01 (not used)
  +- image_02
  | +- 0000
  | | +- 000000.png
  | | +- NNNNNN.png
  | +- 00NN
  |   +- 000000.png
  |   +- NNNNNN.png
  +- image_03
    +- same as image_02
```


## Assumptions

This package assumes that you have also downloaded the calibration data associated with the sequences you want to work on (these are separate files from the sequences themselves).

```python
from wernstrom import get_data

# download the zip data to current directory ... nothing fancy here!
get_data("image_2")
```

## Notation

Homogeneous coordinate transformations are provided as 4x4 `numpy.array` objects and are denoted as `T_destinationFrame_originFrame`.

Pinhole camera intrinsics for camera `N` are provided as 3x3 `numpy.array` objects and are denoted as `K_camN`. Stereo pair baselines are given in meters as `b_gray` for the monochrome stereo pair (`cam0` and `cam1`), and `b_rgb` for the color stereo pair (`cam2` and `cam3`).

## Example

Camera data is available via generators for easy sequential access (e.g., for visual odometry), and by indexed getter methods for random access (e.g., for deep learning). Images are loaded as `PIL.Image` objects using Pillow.

```python
import wernstrom as pykitti

basedir = '/your/dataset/dir'
sequence = '19'

# The 'frames' argument is optional - default: None, which loads the whole dataset.
# Calibration, timestamps, and IMU data are read automatically.
# Camera and velodyne data are available via properties that create generators
# when accessed, or through getter methods that provide random access.
data = pykitti.raw(basedir, sequence, frames=range(0, 50, 5))

# dataset.calib:         Calibration data are accessible as a named tuple
# dataset.timestamps:    Timestamps are parsed into a list of datetime objects
# dataset.oxts:          List of OXTS packets and 6-dof poses as named tuples
# dataset.camN:          Returns a generator that loads individual images from camera N
# dataset.get_camN(idx): Returns the image from camera N at idx
# dataset.gray:          Returns a generator that loads monochrome stereo pairs (cam0, cam1)
# dataset.get_gray(idx): Returns the monochrome stereo pair at idx
# dataset.rgb:           Returns a generator that loads RGB stereo pairs (cam2, cam3)
# dataset.get_rgb(idx):  Returns the RGB stereo pair at idx
# dataset.position:      Returns an array of x,y positions in meters

point_cam0 = data.calib.T_cam0_velo.dot(point_velo)

point_imu = np.array([0,0,0,1])
point_w = [o.T_w_imu.dot(point_imu) for o in data.oxts]

for cam0_image in data.cam0:
    # do something
    pass

cam2_image, cam3_image = data.get_rgb(3)
```

### Transforms `T_w_imu`

"The T_w_imu convention refers to the transformation from IMU to the world coordinate system (so notation is T_destinationFrame_originFrame)." [ref](https://aws.amazon.com/blogs/machine-learning/labeling-data-for-3d-object-tracking-and-sensor-fusion-in-amazon-sagemaker-ground-truth/)

### OpenCV

PIL Image data can be converted to an OpenCV-friendly format using numpy and `cv2.cvtColor`:

```python
img_np = np.array(img)
img_cv2 = cv2.cvtColor(img_np, cv2.COLOR_RGB2BGR)
```

Note: This package does not actually require that OpenCV be installed on your system, except to run `demo_raw_cv2.py`.

## References

- [1] A. Geiger, P. Lenz, C. Stiller, and R. Urtasun, "Vision meets robotics: The KITTI dataset," Int. J. Robot. Research (IJRR), vol. 32, no. 11, pp. 1231–1237, Sep. 2013. [http://www.cvlibs.net/datasets/kitti/](http://www.cvlibs.net/datasets/kitti/) [pdf](docs/Geiger2013IJRR.pdf)
- [raw data format](docs/raw_data_format.md)

# MIT License

**Copyright (c) 2020 Kevin J. Walchko**

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
`
