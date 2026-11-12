# opencv_dnn_object_detector_ros

Bu paket, OpenCV DNN modülü ile **YOLO (Darknet)** ve **TensorFlow SSD MobileNet** modellerini kullanarak kamera üzerinden insan / nesne algılama yapar. ROS Melodic altında çalışır.

## Gereksinimler

- [ROS Melodic](http://wiki.ros.org/melodic/Installation/Ubuntu)
- [OpenCV 4.5.4](https://docs.opencv.org/4.x/d7/d9f/tutorial_linux_install.html)
- Model dosyaları (`cfg`, `weights`, `coco.names`) — repoda yer almaz; yaml içindeki yollar kendi makinenize göre ayarlanmalıdır.

## Nasıl kurulur?

İlk önce bir ROS çalışma ortamı, ardından paket kurulumu yapılmalıdır.

```bash
# ROS calisma ortami olusturma
mkdir -p ~/opencv_dnn_ws/src
cd ~/opencv_dnn_ws
catkin build

# .bashrc dosyasina setup.bash ekleyin (USERNAME yerine kendi kullanici adiniz)
source /home/USERNAME/opencv_dnn_ws/devel/setup.bash

# paketi src altina koyun, sonra derleyin
cd ~/opencv_dnn_ws/src
# Branch kontrolu: "-b f_melodic_init"
. ~/opencv_dnn_ws/devel/setup.bash
cd ~/opencv_dnn_ws && catkin build
```

## Parametre ayarlamak

`config` klasörü altındaki [opencv_dnn_object_detector_ros.yaml](opencv_dnn_object_detector_ros/config/opencv_dnn_object_detector_ros.yaml) içinde istenilen parametrelerin değerleri değiştirilebilir. Koda müdahale edilmeden parametre eklenme**me**li veya çıkarılma**ma**lıdır. Farklı yapıda bir parametre dosyası kullanılmak istenirse `config` içinde yeni bir tane oluşturulabilir ve bu dosya [opencv_dnn_object_detector_ros.launch](opencv_dnn_object_detector_ros/launch/opencv_dnn_object_detector_ros.launch) içindeki `rosparam file` kısmında güncellenmelidir.

### Parametreler

| Parametre | Açıklama |
| --- | --- |
| `formatName` | `tensorflow` veya `darknet` |
| `directoryPathOfCOCONames` | `coco.names` dosya yolu |
| `CONFIDENCE_THRESHOLD` | Varsayılan `0.5` |
| `NMS_THRESHOLD` | Varsayılan `0.4` |
| `NUM_CLASSES` | Varsayılan `80` (`coco.names` sınıf sayısı) |
| `stopDetection` | Varsayılan `false`; algılamayı durdurmak için kullanılır |

**darknet**

- `inputWidthYOLO` / `inputHeightYOLO`: Varsayılan `416`
- `directoryOfPathYOLOConfigFile`: YOLO `.cfg` yolu
- `directoryOfPathYOLOWeights`: YOLO `.weights` yolu

**tensorflow**

- `inputWidthTensorflow` / `inputHeightTensorflow`: Varsayılan `300`
- `directoryOfPathfrozenInferenceGraph`: `.pb` dosya yolu
- `directoryOfPathmodelTensorflowPbtxt`: `.pbtxt` dosya yolu

Örnek yaml:

```yaml
formatName: tensorflow

directoryPathOfCOCONames: /home/USERNAME/yolov3_ws/src/opencv_dnn_object_detector_ros/input/data/coco.names

CONFIDENCE_THRESHOLD: 0.5

NMS_THRESHOLD: 0.4

NUM_CLASSES: 80

stopDetection: false

darknet:
  inputWidthYOLO: 416
  inputHeightYOLO: 416
  directoryOfPathYOLOConfigFile: /home/USERNAME/yolov3_ws/src/opencv_dnn_object_detector_ros/input/cfg/yolov4-tiny.cfg
  directoryOfPathYOLOWeights: /home/USERNAME/yolov3_ws/src/opencv_dnn_object_detector_ros/input/weights/yolov4-tiny.weights

tensorflow:
  inputWidthTensorflow: 300
  inputHeightTensorflow: 300
  directoryOfPathfrozenInferenceGraph: /home/USERNAME/yolov3_ws/src/opencv_dnn_object_detector_ros/input/weights/frozen_inference_graph.pb
  directoryOfPathmodelTensorflowPbtxt: /home/USERNAME/yolov3_ws/src/opencv_dnn_object_detector_ros/input/cfg/ssd_mobilenet_v2.pbtxt
```

> Not: Mevcut config dosyasındaki yollar `/home/plnm20/...` makinesine göredir. Kendi ortamınızda bu yolları güncelleyin. `input/` altında model dosyaları yoksa paket çalışmaz.

## Nasıl çalıştırılır?

Kurulumdan sonra:

```bash
cd ~/opencv_dnn_ws/
roslaunch opencv_dnn_object_detector_ros opencv_dnn_object_detector_ros.launch
```

Kamera açılır ve seçilen modele göre sonsuz döngüde tespit yapılır. ESC ile görüntü penceresi kapatılır; ardından `CTRL+C` ile node durdurulur.

## CMakeLists.txt

Önemli nokta: `find_package(OpenCV 4.5.4 REQUIRED)`. Sürüm belirtilmezse ROS ile gelen OpenCV kullanılır.

```cmake
cmake_minimum_required(VERSION 3.0.2)
project(opencv_dnn_object_detector_ros)
find_package(OpenCV 4.5.4 REQUIRED)
find_package(catkin REQUIRED COMPONENTS
  roscpp
  rospy
  std_msgs
  roslib
)
```

## Kaynak kod akışı

Ana dosya: `opencv_dnn_object_detector_ros/src/opencv_dnn_object_detector_ros.cpp`

### Başlatma sırası (`initialization`)

1. `readFromYAMLFile()` — ROS parametrelerini okur
2. `readFilesYOLOV3()` — `coco.names` sınıf isimlerini yükler
3. `setupNet()` — `formatName`'e göre Darknet veya TensorFlow ağını kurar
4. `openVideoCaptureStream()` — kamera (`0`) açılır ve ilgili sonsuz döngü başlar

### `readFromYAMLFile()`

`ros::NodeHandle` ile yaml parametrelerini okur (`formatName`, eşikler, model yolları).

### `readFilesYOLOV3()`

`directoryPathOfCOCONames` dosyasındaki sınıf isimlerini `classNames` vektörüne ekler.

### `setupNet()`

- `darknet` → `cv::dnn::readNetFromDarknet(...)`
- `tensorflow` → `cv::dnn::readNet(..., "TensorFlow")`

### `detectDarknet(const cv::Mat &frame)`

- Giriş: görüntü matrisi
- Çıkış: algılanan kutular (`std::vector<cv::Rect>`)
- Yalnızca `person` sınıfı için kutu çizer
- `stopDetection` ile algılama açılıp kapatılabilir
- FPS ve tespit sayısı ekrana yazılır

### `detectTensorflow(const cv::Mat &frame)`

- Giriş: görüntü matrisi
- Çıkış: algılanan kutular (`std::vector<cv::Rect>`)
- Eşik üstü tüm sınıflar için kutu ve etiket çizer

### `cleanup()` / `reset(bool open)`

- `cleanup`: `stopDetection = true`, kamera ve pencereler kapatılır
- `reset`: önce `cleanup`, sonra `stopDetection = false`; `open == true` ise `initialization()` yeniden çağrılır

## Paket yapısı

```
opencv_dnn_object_detector_ros/
  CMakeLists.txt
  package.xml
  config/opencv_dnn_object_detector_ros.yaml
  include/opencv_dnn_object_detector_ros.hpp
  launch/opencv_dnn_object_detector_ros.launch
  src/opencv_dnn_object_detector_ros.cpp
  input/   # model dosyalari buraya konmali (cfg, weights, data)
```
