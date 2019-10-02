<<<<<<< HEAD
This is a repository with example worlds generated from real world examples.

Here are some notes 


# Download content

Go to 
https://earthexplorer.usgs.gov/

Create an account

Find the region of interest. Select a point or set of points.


Select Data Set(s):

 * Aerial Imagry -> High Resolution Orthoimagery
 * Digital Elevation -> SRTM -> SRTM 1 Arc-Second Global

Go to the Results and download all the tiles.
You can download them individually or in a bulk download.

Alternative data sources should be fine as well.

There will be a lot of data.
The 9 tiles I used for McMillan were each approximately 400MB
And the height map arc tile is ~ 25MB.


# Isolate area of interest

Once you have the region of interest use gdalwarp to extract the regions of interest.
This is the script I used for mcmillan and yosemite


    #!/bin/bash                                                                     

    set -x

    # Yosemite                                                                      
    SOUTH=37.6993
    NORTH=37.7690
    EAST=-119.5285
    WEST=-119.5978
    NAME=yosemite
    RESOLUTION=5000
    SOURCE_DIR=/data/temp_gis
    PACKAGE_DIR=/tmp/ws/src/uav_testing/yosemite_valley
    SOURCE_DEM=n37_w120_1arc_v3.tif
    SOURCE_IMAGES='m_3711912_se_11_h_20160701.tif m_3711912_sw_11_h_20160701.tif m_\
    3711920_ne_11_h_20160701.tif m_3711920_nw_11_h_20160701.tif'

    # mcmillan                                                                      
    SOUTH=35.692125
    NORTH=35.755904
    EAST=-120.735120
    WEST=-120.798966
    NAME=mcmillan
    RESOLUTION=5000
    SOURCE_DIR=/data/temp_gis/mcmillan
    PACKAGE_DIR=/tmp/ws/src/uav_testing/mcmillan_airfield
    SOURCE_DEM=n35_w121_1arc_v3.tif
    SOURCE_IMAGES='5724_2452.tif 5724_2462.tif 5724_2472.tif 5734_2452.tif 5734_246\
    2.tif 5734_2472.tif 5744_2452.tif 5744_2462.tif 5744_2472.tif'


    OUTPUT_IMAGE=${NAME}_color.tif
    OUTPUT_DEM=${PACKAGE_DIR}/media/${NAME}_elevation.tif
    OUTPUT_PNG=${PACKAGE_DIR}/media/textures/${NAME}_color.png


    cd $SOURCE_DIR

    rm -f $OUTPUT_DEM $OUTPUT_IMAGE $OUTPUT_PNG


    gdalwarp -te $WEST $SOUTH $EAST $NORTH $SOURCE_DEM  $OUTPUT_DEM

    gdalwarp -t_srs '+proj=longlat +datum=WGS84 +no_defs ' -ts $RESOLUTION 0 -te $W\
    EST $SOUTH $EAST $NORTH $SOURCE_IMAGES $OUTPUT_IMAGE
    convert $OUTPUT_IMAGE $OUTPUT_PNG



Save these files into your gazebo resource path that's exported by the package.

Note that the 5000 is the resolution of the file laterally.

# Generate World

Then create a world that references them.

I pushed the pose down such that the airfield is at zero height in gazebo.
The size of the texture is the width of the content in meters.

Note that there's a scaling issue with the textures. I have ticketed it here: https://bitbucket.org/osrf/gazebo/issues/2603/texture-scaling-on-heightmaps-does-not

It looks like a rendering of about 80% width is necessary.


Gazebo also seems to work better with square content, this might have been an artifact from before validating the above ratio.

# Developmet tip

If you are changing any textures on the height map.
Whether content or meta data it is all cached.
To change anything you will need to remove the cached paging content in `~/.gazebo/paging/VISUAL_ELEMENT`

Ticketed at: https://bitbucket.org/osrf/gazebo/issues/2604/the-default-caching-behavior-of-caching



# Resources:

A potential other approach is to overlay in QGIS: https://opengislab.com/blog/2018/3/20/3d-dem-visualization-in-qgis-30

gdalwarp docs: https://www.gdal.org/gdalwarp.html

SDF geometry documentation: http://sdformat.org/spec?elem=geometry&ver=1.6
=======
# Gazebo for MAVLink SITL and HITL [![Build Status](https://travis-ci.org/PX4/sitl_gazebo.svg?branch=master)](https://travis-ci.org/PX4/sitl_gazebo)

This is a flight simulator for multirotors, VTOL and fixed wing. It uses the motor model and other pieces from the RotorS simulator, but in contrast to RotorS has no dependency on ROS. This repository is in the process of being re-integrated into RotorS, which then will support ROS and MAVLink as transport options: https://github.com/ethz-asl/rotors_simulator

**If you use this simulator in academic work, please cite RotorS as per the README in the above link.**

## Install Gazebo Simulator

Follow instructions on the [official site](http://gazebosim.org/tutorials?cat=install) to install Gazebo. Mac OS and Linux users should install Gazebo 7.


## Protobuf

Install the protobuf library, which is used as interface to Gazebo.

### Ubuntu Linux

```bash
sudo apt-get install libprotobuf-dev libprotoc-dev protobuf-compiler libeigen3-dev \
			gazebo7 libgazebo7-dev libxml2-utils python-rospkg python-jinja2
```

### Mac OS

```bash
pip install rospkg jinja2
brew install graphviz libxml2 sdformat3 eigen opencv
brew install gazebo7
```

An older version of protobuf (`< 3.0.0`) is required on Mac OS:

```bash
brew tap homebrew/versions
brew install homebrew/versions/protobuf260
```

## Build Gazebo Plugins (all operating systems)

Clone the gazebo plugins repository to your computer. IMPORTANT: If you do not clone to ~/src/sitl_gazebo, all remaining paths in these instructions will need to be adjusted.

```bash
mkdir -p ~/src
cd src
git clone --recursive https://github.com/PX4/sitl_gazebo.git
```

Create a build folder in the top level of your repository:

```bash
mkdir Build
```

Next add the location of this build directory to your gazebo plugin path, e.g. add the following line to your .bashrc (Linux) or .bash_profile (Mac) file:


```bash
# Set the plugin path so Gazebo finds our model and sim
export GAZEBO_PLUGIN_PATH=${GAZEBO_PLUGIN_PATH}:$HOME/src/sitl_gazebo/Build
# Set the model path so Gazebo finds the airframes
export GAZEBO_MODEL_PATH=${GAZEBO_MODEL_PATH}:$HOME/src/sitl_gazebo/models
# Disable online model lookup since this is quite experimental and unstable
export GAZEBO_MODEL_DATABASE_URI=""
```

You also need to add the the root location of this repository, e.g. add the following line to your .bashrc (Linux) or .bash_profile (Mac) file:
```bash
# Set path to sitl_gazebo repository
export SITL_GAZEBO_PATH=$HOME/src/sitl_gazebo
```

Navigate into the build directory and invoke CMake from it:

```bash
cd ~/src/sitl_gazebo
cd Build
cmake ..
```

Now build the gazebo plugins by typing:

```bash
make
```

### GStreamer Support
If you want support for the GStreamer camera plugin, make sure to install
GStreamer before running `cmake`. Eg. on Ubuntu with:
```
sudo apt-get install gstreamer1.0-* libgstreamer1.0-*
```

### Geotagging Plugin
If you want to use the geotagging plugin, make sure you have `exiftool`
installed on your system. On Ubuntu it can be installed with:
```
sudo apt-get install libimage-exiftool-perl
```

## Install

If you wish the libraries and models to be usable anywhere on your system without
specifying th paths, install as shown below.

**Note: If you are using ubuntu, it is best to see the packaging section.**

```bash
sudo make install
```

## Testing

Gazebo will now launch when typing 'gazebo' on the shell:

```bash
. /usr/share/gazebo/setup.sh
. /usr/share/mavlink_sitl_gazebo/setup.sh
gazebo worlds/iris.world
```

Please refer to the documentation of the particular flight stack how to run it against this framework, e.g. [PX4](http://dev.px4.io/simulation-gazebo.html)

## Packaging

### Deb

To create a debian package for ubuntu and install it to your system.

```bash
cd Build
cmake ..
make
rm *.deb
cpack -G DEB
sudo dpkg -i *.deb
```
>>>>>>> c3b2a0d5f8a4088d064ca8e0df7c329c98395232
