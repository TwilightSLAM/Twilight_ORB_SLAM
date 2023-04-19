# Twilight_ORB_SLAM3
Main goal: To carry out a comparative study of the effect of different image enhancement modules (i.e. Bread, Dual, EnlightenGAN, Zero-DCE) on the performaNce of ORB-SLAM3 architecture, with monocular camera SLAM dataset sequences.

Experiment Overview: We initially test against against the KITTI benchmark to confirm pipeline success. Then, we test against ETH3D (sfm_house_loop and sfm_lab_room2) data sequences which were qualitatively found to have more underexposed images on average or in general througout the entire sequence. 

# 1. OS and Hardware prerequisites
We tested this project in Ubuntu 20.04. Additionally, the original repository reports testing for Ubuntu Ubuntu 16.04 and 18.04. 

# 2. Installation of ORB-SLAM 3 on a fresh installed Ubuntu 20.04
Install all liberay dependencies.
```shell

sudo apt update

sudo apt-get install build-essential
sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev

sudo apt-get install python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libdc1394-22-dev libjasper-dev

sudo apt-get install libglew-dev libboost-all-dev libssl-dev

sudo apt install libeigen3-dev

### Install OpenCV 3.2.0
The ORB-SLAM 3 was test by  
```shell
cd ~
mkdir Dev && cd Dev
git clone https://github.com/opencv/opencv.git
cd opencv
git checkout 3.2.0
```
Put the following at the top of header file `gedit ./modules/videoio/src/cap_ffmpeg_impl.hpp`  
`#define AV_CODEC_FLAG_GLOBAL_HEADER (1 << 22)`  
`#define CODEC_FLAG_GLOBAL_HEADER AV_CODEC_FLAG_GLOBAL_HEADER`  
`#define AVFMT_RAWPICTURE 0x0020`  
and save and close the file
```shell
mkdir build
cd build
cmake -D CMAKE_BUILD_TYPE=Release -D WITH_CUDA=OFF -D CMAKE_INSTALL_PREFIX=/usr/local ..
make -j 3
sudo make install
```
> If you want to install to conda environment, use `CMAKE_INSTALL_PREFIX=$CONDA_PREFIX` instead.
---

### Install Pangolin
Now, we install the Pangolin. I used the commit version 86eb4975fc4fc8b5d92148c2e370045ae9bf9f5d
```shell
cd ~/Dev
git clone https://github.com/stevenlovegrove/Pangolin.git
cd Pangolin 
mkdir build 
cd build 
cmake .. -D CMAKE_BUILD_TYPE=Release 
make -j 3 
sudo make install
```
> If you want to install to conda environment, add `CMAKE_INSTALL_PREFIX=$CONDA_PREFIX` instead.
---

### ORB-SLAM 3
Now, we install ORB-SLAM3. I used the commit version ef9784101fbd28506b52f233315541ef8ba7af57 tag: v0.3-beta

```shell
cd ~/Dev
git clone https://github.com/UZ-SLAMLab/ORB_SLAM3.git 
cd ORB_SLAM3
```
We need to change the header file `gedit ./include/LoopClosing.h` at line 51  
from  
`Eigen::aligned_allocator<std::pair<const KeyFrame*, g2o::Sim3> > > KeyFrameAndPose;`  
to  
`Eigen::aligned_allocator<std::pair<KeyFrame *const, g2o::Sim3> > > KeyFrameAndPose;`
in order to make this comiple.  
Now, we can comiple ORB-SLAM3 and it dependencies as DBoW2 and g2o.  

Now Simply just run (if you encounter compiler, try to run the this shell script 2 or 3 more time. It works for me.)
```shell
./build.sh
```
to install

# 3. Download test datasets

```shell
cd ~
mkdir -p Datasets/EuRoc
cd Datasets/EuRoc/
wget -c http://robotics.ethz.ch/~asl-datasets/ijrr_euroc_mav_dataset/machine_hall/MH_01_easy/MH_01_easy.zip
mkdir MH01
unzip MH_01_easy.zip -d MH01/

# 3. Run simulation 
```shell
cd ~/Dev/ORB_SLAM3

# Pick of them below that you want to run

# Mono
./Examples/Monocular/mono_euroc ./Vocabulary/ORBvoc.txt ./Examples/Monocular/EuRoC.yaml ~/Datasets/EuRoc/MH01 ./Examples/Monocular/EuRoC_TimeStamps/MH01.txt dataset-MH01_mono

# Mono + Inertial
./Examples/Monocular-Inertial/mono_inertial_euroc ./Vocabulary/ORBvoc.txt ./Examples/Monocular-Inertial/EuRoC.yaml ~/Datasets/EuRoc/MH01 ./Examples/Monocular-Inertial/EuRoC_TimeStamps/MH01.txt dataset-MH01_monoi

# Stereo
./Examples/Stereo/stereo_euroc ./Vocabulary/ORBvoc.txt ./Examples/Stereo/EuRoC.yaml ~/Datasets/EuRoc/MH01 ./Examples/Stereo/EuRoC_TimeStamps/MH01.txt dataset-MH01_stereo

# Stereo + Inertial
./Examples/Stereo-Inertial/stereo_inertial_euroc ./Vocabulary/ORBvoc.txt ./Examples/Stereo-Inertial/EuRoC.yaml ~/Datasets/EuRoc/MH01 ./Examples/Stereo-Inertial/EuRoC_TimeStamps/MH01.txt dataset-MH01_stereoi
```

# 4 Validation Estimate vs Ground True
We need numpy and matplotlib installed in pytho2.7. But Ubuntu20.04 has not pip2.7
```shell
sudo apt install curl
cd ~/Desktop
curl https://bootstrap.pypa.io/2.7/get-pip.py --output get-pip.py
sudo python2 get-pip.py
pip2.7 install numpy matplotlib
```

**Run and plot Ground true**
```
cd ~/Dev/ORB_SLAM3

./Examples/Stereo/stereo_euroc ./Vocabulary/ORBvoc.txt ./Examples/Stereo/EuRoC.yaml ~/Datasets/EuRoc/MH01 ./Examples/Stereo/EuRoC_TimeStamps/MH01.txt dataset-MH01_stereo
```

**Plot estimate vs Ground true**
```
cd ~/Dev/ORB_SLAM3

python evaluation/evaluate_ate_scale.py evaluation/Ground_truth/EuRoC_left_cam/MH01_GT.txt f_dataset-MH01_stereo.txt --plot MH01_stereo.pdf
