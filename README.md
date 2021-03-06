# PyRealsense for d435

ready-to-use exe version can be seen at https://github.com/soarwing52/Realsense_exe

in this project will have record bag and measure picture from bag with the RealsenseD435 camera. The scripts are separated into two parts, collector and processor.

the collector is to let the driver collect data effortlessly, with one remote controller.

the processor is creating data from the bag files and the txt recorded, match frames and location, using arcpy API to create shapefile for users.

The further usage is using matplotlib within ArcGIS 10, let users read ROSbag files with hyperlink script function 

## Getting Started

the script is mainly based on Realsense, using ROSbag and txt files to record gps location

*the save_single_frame function can't get depth data, show always 0,0,0, therefore I can only use rs.recorder


### Prerequisites

Hardware: GPS reciever (BU353S4 in my case)
![gps](http://village.photos/images/user/2222e0fa-3e6d-47fa-8ee0-19f1dc512885/ddbe9b26-bc4b-48e7-b669-8f25ad00a494.jpg)

RS camera
![RS](https://simplecore.intel.com/realsensehub/wp-content/uploads/sites/63/Compare.png)



### Data Preparation

data collector: will generate folders: bag, log, and csvs :live location. foto_location

data processor: from the collected bag and log file, create jpg and matched.txt


### Installing

A step by step series of examples that tell you how to get a development env running

will generate requirment.txt in the future
Python packages
```
pip install --upgrade pip
pip install --user pyrealsense2 
pip install --user opencv-python (especially in user, else it will overwrite arcgis python package)
argparse, py-getch, pyserial, fractions,
```

### Data Capture

the data capture script will automatically generate three folders: bag, foto_log

it will put the recorded bag in bag format, 

information of index,color frame number, depth frame number, longtitude, lattitude, day, month, year, timewill be saved in txt format in foto_log

QtCam:

![qtcam](https://1.bp.blogspot.com/-4gsC0uT-b4o/XbwRcqa1I5I/AAAAAAABAgw/DfpxRpPVBPQQbllyI8gTpV_EIy2GAsAegCLcBGAsYHQ/s400/pyqtcam.JPG)


The new version:
now it will write two csv files, and is used to open in QGIS, set open layer with csv, and the rendering frequency 1-5 seconds 
depends on the performance of your hardware.
The reason is QGIS can render the huge amount of point data better than Google Earth

in QGIS:
Add Delimited Text Layer and set render frequency to 1 second for live.csv, foto_location.csv

------------------------------------------------------------------------------------------------------------------
kml will generate live location as live, and the location with recorded places in Foto.kml
use Google Earth: Temporary Places/Add/Networklink link to folder/kml/ live for location Foto.kml for foto point

![g earth](https://github.com/soarwing52/RealsensePython/blob/master/examples/google%20earth.PNG?raw=true)

### Data process

to prepare the data, use the data_prepare.py

click the three buttons in the order left to right

1.generate jpg
using openCV to generate JPEG files, one sharpening filter is added.
```
                kernel = np.array([[-1, -1, -1],
                                   [-1, 9, -1],
                                   [-1, -1, -1]])
```
*due to disk writing speed, sometimes the realsense api will miss some frames, this script loops through the bag files for two times. can be run multiple times if needed

*this script will skip existed files

2.generate shp
because of latency, the frames recorded and the frame number in foto_log can jump in a range around +-2, for example: in log is frame no.100 but it could happen that the recorded is 99-102, so one extra step to match the real frame number to the log file is needed.

after the matcher.txt is generated, this is the complete list of points taken, use MakeXYEventLayer, and FeatureClassToShapefile_conversion will save a shapefile.

*the earlier version has match the frame in a 25ms timestamp for Color and Depth to make sure the frames are matching, the current version removed it, since the latency is acceptable

3.geotag

according to the matcher.txt, photos EXIF will be written, the Lontitude and Latitude

### in ArcGIS

in Arcgis use hyperlink and show script, select python
put in 

```
import subprocess
def OpenLink ( [jpg_path] ):
  bag = [jpg_path] 
  comnd = 'measure.exe -p {}'.format(bag)
  subprocess.call(comnd)
  return
```


The method I suggest here is to use the hlk file, by open hlk from the arcgis integration folder, the working directory will be automatically changed to this folder. 
Then the video, photo function can be open automatically.

End with an example of getting some data out of the system or using it for a little demo

in here we use subprocess because the current realsense library will freeze after set_real_time(False)
and ArcMap don't support multithread nor multicore
therefore we can't use the simple import but have to call out command line and then run python

The running image will be like this:

use spacebar to switch between modes, video mode can browse through the frames faster
left click and hold to measure, right click to erase all
![videomode](https://github.com/soarwing52/Realsense_exe/blob/master/img/video%20mode.JPG)
![measure_mode](https://github.com/soarwing52/Realsense_exe/raw/master/img/measure%20mode.JPG)

## Authors

* **Che-Yi Hung** - *Initial work* - [soarwing52](https://github.com/soarwing52)

