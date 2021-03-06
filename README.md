**A fun IoT app using a Raspberry Pi + camera.**
The app detects [motion from the h264 encoder](http://picamera.readthedocs.io/en/release-1.12/recipes2.html#recording-motion-vector-data) with little CPU drain. A first snapshot is taken once the total motion in the video stream exceeds a certain threshold. A second snapshot is taken after the scene becomes static again. Finally, the second snapshot is analyzed. Thus, this _Thing of the Internet_ is a (wonky) surveillance camera and a selfie-machine at the same time --however you want to view it. The purpose was to demo Azure IoT and cognitive services on top of building an image acquisition framework for the RPi.

## Automatic Image Captioning
The importance of (data) privacy grows daily, but having a NN talk about its observations might just be ok... Thus, snapshots get only persisted on the local file system. The gist of the second snapshot is extracted by Microsoft's [computer vision API](https://www.microsoft.com/cognitive-services/en-us/computer-vision-api). This gist consists of tags, categories and a caption from the second snapshot. It is passed on to the cloud.
## IoT Telemetry:
Next to the description and other features at the end of the scene, telemetry includes motion vectors for each frame during a scene. Learning gestures from this dataset would be even more fun! I wanted to try Azure's IoT Hub for data ingestion. All data mentioned above is forwarded via device-to-cloud-messages.

# Example
**I'm entering the living room from the left**
![alt-text](https://raw.githubusercontent.com/ahirner/room-glimpse/master/example_snapshots/2017-02-21T10_52_21.227244_on.jpg)

The motion detector triggers the first snapshot to be stored on the RPi. At the same time, motion vector data from each video frame is forwarded to the cloud asynchronously.

**I pause to complete the scene**   
![alt-text](https://raw.githubusercontent.com/ahirner/room-glimpse/master/example_snapshots/2017-02-21T10_52_22.553099_off.jpg)

```javascript
    caption: 'a man that is standing in the living room'
    confidence: 0.1240666986256891
    tags: 'floor', 'indoor', 'person', 'window', 'table', 'room', 'man', 'living', 'holding', 'young', 'black', 'standing', 'woman', 'dog', 'kitchen', 'remote', 'playing', 'white'
```
This is how the second snapshot is described by Azure's cognitive API. Fair enough... Unfortunately, the caption doesn't mention my awesome guitar performance. The description of the scene and meta-information like timestamps are dispatched whereas recording motion-data stops.

**I leave the room after much applause** :clap::clap::clap: (snapshot omitted)...

After no motion was detected for a set amount of time (0.75 secs in that case), another scene is analyzed.

**Now it's just the the bare room**   
![alt-text](https://raw.githubusercontent.com/ahirner/room-glimpse/master/example_snapshots/2017-02-21T10_52_24.915270_off.jpg)
```javascript
    caption: 'a living room with hard wood floor'
    confidence: 0.9247661343688557
    tags: 'floor', 'indoor', 'room', 'living', 'table', 'building', 'window', 'wood', 'hard', 'wooden', 'sitting', 'television', 'black', 'furniture', 'kitchen', 'small', 'large', 'open', 'area', 'computer', 'view', 'home', 'white', 'modern', 'door', 'screen', 'desk', 'laptop', 'dog', 'refrigerator', 'bedroom'
```
This time, the description is pretty accurate (and confident).

# Installation
- [Setup](https://azure.microsoft.com/en-us/resources/samples/iot-hub-c-raspberrypi-getstartedkit/) an Azure IoT Hub and add the RPi as a device.
- `git clone https://github.com/ahirner/room-glimpse.git`
- Create `credentials.py` in `./creds` with the Azure Cognitive API key, the IoT device ID and a device connection string.
```python
AZURE_COG_KEY= 'xxx'
AZURE_DEV_ID= 'yyy'
AZURE_DEV_CONNECTION_STRING='HostName=zzz.azure-devices.net;SharedAccessKeyName=zzz;SharedAccessKey=zzz='
```
- Install missing modules (_`requirements.txt` tbd_) 
- Start with `python3 room-glimpse.py`

Only the HTTP API is used for now. The dedicated [azure-iot-python SDK](https://github.com/azure/azure-iot-sdk-python) can [control batching more effectively](https://github.com/Azure/azure-iot-sdk-python/issues/15), use MQTT for less overhead but is not yet available via pip3 on Unix.

Configuration for the video stream, motion thresholds and cloud endpoints are in `config.py`.

# More ideas
1) Of course, nothing prevents you from running/training your own version of a [talking NN](https://github.com/tensorflow/models/tree/master/im2txt). In fact, this project is a vantage point to try pushing computing on the edge. Sam maintains a [pip wheel to install TensorFlow](https://github.com/samjabrahams/tensorflow-on-raspberry-pi) on the little RPi. [Pete Warden](https://petewarden.com/2016/12/30/rewriting-tensorflow-graphs-with-the-gtt/) has done amazing work recently to trim down NNs in a principled way (e.g. quantization for fixed point math).

2) In general, make use of spare cores. Most of the time, the CPU idles at 15% (remember the h264 motion detection). So there is plenty of room left for [beefier tasks](http://www.pyimagesearch.com/2016/04/18/install-guide-raspberry-pi-3-raspbian-jessie-opencv-3/) on the edge. 

3) Overlay motion vectors in a live web view (there is a 2D vector for each 16x16 macro block).
