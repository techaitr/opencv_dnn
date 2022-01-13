# opencv_dnn_object_detector_ros

Bu paket,"YOLO" ve "Tensorflow SSD mobile net" modellerini kullanarak insan algılama için kullanılır.

## Gereksinimler

[ROS-Melodic](http://wiki.ros.org/melodic/Installation/Ubuntu)

[OpenCV 4.5.4](https://docs.opencv.org/4.x/d7/d9f/tutorial_linux_install.html)

## Nasıl kurulur?

İlk önce bir ros çalışma ortami ardındanda ros paketi oluşturmalısınız.

```bash
#ros calisma ortami oluşturma.
mkdir -p ~/opencv_dnn_ws/src
# workspace gidilir.
cd opencv_dnn_ws 
# workspace in derlenmesi gerekir. 
catkin build
# .bashrc dosyasina aşagıdaki gibi setup.bash eklemelisin.
source /home/USERNAME/opencv_dnn_ws/devel/setup.bash
#ros pkg olusturma...
cd src/
git clone ssh://git@192.168.100.2:350/********.git 
# Branch kontrolü yapabilirsiniz...  "-b f_melodic_init"
. ~/opencv_dnn_ws/devel/setup.bash
# derlemek icin workspace gidilir ve derlenir 
cd ~/opencv_dnn_ws/src && catkin build
```

## Parametre ayarlamak

`config` klasörü altındaki [opencv_dnn_object_detector_ros.yaml](config/opencv_dnn_object_detector_ros.yaml)  içinde istenilen parametrelerin değerleri değiştirilebilir. Koda müdahale edilmeden parametre eklenme**me**li veya çıkarılma**ma**lıdır. Farklı yapıda bir parametre dosyası kullanılmak istenirse uygun `config` içinde yeni bir tane oluşturulabilir ve bu dosya  [opencv_dnn_object_detector_ros.launch](launch/opencv_dnn_object_detector_ros.launch) içinde rosparam file kısmında güncellenmelidir.  Detaylar, ilgili başlıklar altında yer almaktadır. Parametreler, isimleri, seçim sınırları ve kullanım tavisyeleri aşağıdadır. 

### Parametreler:

`formatName`:Parametre olarak `tensorflow` veya `darknet` değerlerini alır.

`directoryPathOfCOCONames`:Parametre olarak  `coco.names`'in dosya yolunu alır. Örneğin: **/home/USERNAME/yolov3_ws/src/opencv_dnn_object_detector_ros/input/data/coco.names**

`CONFIDENCE_THRESHOLD`: Parametre olarak **0.5** değeri default değerdir.

`NMS_THRESHOLD`: Parametre olarak **0.4** değeri default değeridir.

`NUM_CLASSES`: Parametre olarak **80**'dir. `coco.names` içindeki sınıf sayısıdır.

`stopDetection`: Parametre olarak **false** ile başlatılır. İnsan algılama durdurmak için kullanılır. 

`darknet`: 

	`  inputWidthYOLO`: Parametre olarak kullanılan `YOLO` modelinin görüntü `Width` boyutu verilir. Default olarak 416'dır.
	
	`  inputHeightYOLO`: Parametre olarak kullanılan `YOLO` modelinin görüntü `Height` boyutu verilir. Default olarak 416'dır.
	
	`  directoryOfPathYOLOConfigFile`: Parametre olarak `YOLO` modelinin `cfg` dosyasının uzantısı verilir. **/home/USERNAME/yolov3_ws/src/opencv_dnn_object_detector_ros/input/cfg/yolov4-tiny.cfg**
	
	` directoryOfPathYOLOWeights`: Parametre olarak `YOLO` modelin ağırlık dosyasının uzantısı verilir. /home/USERNAME/yolov3_ws/src/opencv_dnn_object_detector_ros/input/weights/yolov4-tiny.weights
	
`tensorflow`: Parametre olarak  

	`  inputWidthTensorflow`: Parametre olarak kullanılan `YOLO` modelinin görüntü `Width` boyutu verilir. Default olarak 300'dür.
	
	`  inputHeightTensorflow`: Parametre olarak kullanılan `YOLO` modelinin görüntü `Height` boyutu verilir. Default olarak 300'dür.
	
	`  directoryOfPathfrozenInferenceGraph`:Parametre olarak **/home/USERNAME/yolov3_ws/src/opencv_dnn_object_detector_ros/input/weights/frozen_inference_graph.pb** .pb uzantılı tensorflow uzantılı dosya yolunu alır.
	
	`  directoryOfPathmodelTensorflowPbtxt`: Parametre olarak **/home/USERNAME/yolov3_ws/src/opencv_dnn_object_detector_ros/input/cfg/ssd_mobilenet_v2.pbtxt** .pbtxt uzantılı dosya yolunu alır.
	

~~~
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
  
~~~

## Nasıl Çalıştırılır?

[Kurulum](## Nasıl Kurulur?) yapıldıktan sonra;

```bash
cd ~/opencv_dnn_ws/ 
roslaunch opencv_dnn_object_detector_ros opencv_dnn_object_detector_ros.launch
```

Kod çalıştırılmış olur. Kapatılmak istenirse `ESC` basılır ve görüntü penceresi kapatırlır ardından `CTRL+C` ile kod durdurulur.

## CMakeLists.txt

CMakeLists.txt dosyası aşağıdaki gibi yazılmıştır.

Burada önemli olan şeylerden birisi `find_package(OpenCV 4.5.4 REQUIRED)`'dir.  Eğer ki OpenCV sürümünü belirtmezsek `ROS` ile gelen versiyonu kullanılacaktır. Bunu engellemek için OpenCV 4.5.4 versiyonunu belirtiyoruz. Eğer başka bir projede OpenCV'nin bu 4.5.4 versiyonunu kullanmak istemezsek.

```bash
cmake_minimum_required(VERSION 3.0.2)
project(opencv_dnn_object_detector_ros)
find_package(OpenCV 4.5.4 REQUIRED)
find_package(catkin REQUIRED COMPONENTS
  roscpp
  rospy
  std_msgs
  roslib
)
catkin_package(
  INCLUDE_DIRS include
  LIBRARIES opencv_dnn_object_detector_ros
  CATKIN_DEPENDS roscpp rospy std_msgs
  #DEPENDS system_lib
)
include_directories(
  include
  ${catkin_INCLUDE_DIRS}
)
add_executable(opencv_dnn_object_detector_ros src/opencv_dnn_object_detector_ros.cpp)
target_link_libraries(opencv_dnn_object_detector_ros  
	${catkin_LIBRARIES}  
	${OpenCV_LIBRARIES}
    opencv_objdetect
    opencv_core 
    opencv_imgproc
    opencv_highgui
    opencv_imgproc
    opencv_objdetect
    opencv_videoio
)
```

## Kaynak Kod Dosyaları

Dosyalar ve içlerindeki sınıflar ile ilgili bilgiler aşağıda yer almaktadır.

### opencv_dnn_object_detector_ros.cpp

* OpenCV DNN modülünü kullanarak tespit işleminin yapıldığı `cpp` dosyasıdır.

#### bool opencv_dnn_object_detector_ros::initialization()

* Kodun başlatılması için çağrılması gereken fonksiyondur.
* Fonksiyon çağrıldırıldığında sırasıyla; `readFromYAMLFile`, `setupNet` fonksiyonkları çağrılır.

~~~
    if (readFromYAMLFile())
    {
        packagePath = ros::package::getPath("opencv_dnn_object_detector_ros");
        std::cout << "packagePath: " << packagePath << std::endl;
        if (readClassNameFromNamesFile())
        {
            if (setupNet())
            {
               return true;
            }
            else
            {
                return false;
            }
        }
        else
        {
            return false;
        }
    }
    else
    {
        return false;
    }

~~~

#### bool opencv_dnn_object_detector_ros::readFromYAMLFile()

* `ros::NodeHandle nh;`  ifadesi tanımlanır ve bununla `YAML` dosyasından okuma işlemleri aşağıdaki gibi yapılır.

```	nh.getParam("/formatName", formatName);
nh.getParam("/directoryPathOfCOCONames", directoryPathOfCOCONames);
nh.getParam("/CONFIDENCE_THRESHOLD", CONFIDENCE_THRESHOLD);
nh.getParam("/NMS_THRESHOLD", NMS_THRESHOLD);
nh.getParam("/NUM_CLASSES", NUM_CLASSES);
```

 

####  bool opencv_dnn_object_detector_ros::readClassNameFromNamesFile()

* `coco.names` dosyasındaki obje isimlerini okur ve `std::vector<std::string>` formatındaki `classNames` değişkene atar.

```	
std::ifstream class_file(directoryPathOfCOCONames);
if (!class_file)
{
	std::cerr << "failed to open" << directoryPathOfCOCONames << "\n";
	return false;
}
while (getline(class_file, line))
{
	classNames.push_back(line);
}

```

#### bool opencv_dnn_object_detector_ros::setupNet()

* Bu fonksiyonda `YAML` dosyasındaki `formatName`'e göre `darknet` veya `tensorflow` modellerine göre kod yapısı değişmektedir. 

  ```
  if (formatName == "darknet")
  {
  	net = cv::dnn::readNetFromDarknet(directoryOfPathYOLOConfigFile, directoryOfPathYOLOWeights);
  	outputNames = net.getUnconnectedOutLayersNames();
  	return true;
  }
  else if (formatName == "tensorflow")
  {
  	/* code */
  	net = cv::dnn::readNet(directoryOfPathfrozenInferenceGraph, directoryOfPathmodelTensorflowPbtxt, "TensorFlow");
  	outputNames = net.getUnconnectedOutLayersNames();
  	return true;
  }
  
  ```



#### std::vector<<cv::Rect>> opencv_dnn_object_detector_ros::detectDarknet(const cv::Mat &frame)

* Fonksiyon girdisi görüntü matrisi referansıdır. 
* Çıktı ise `vector` tipinde OpenCV'nin `rectangular` formatındadır ve algılanan nesnelerin görüntü üzerindeki koordinat bilgilerini verir.
* `stopDetection` ile algılama işlemi ile açma-kapama işlemi gerçekleştirilir.



#### std::vector<cv::Rect> opencv_dnn_object_detector_ros::detectTensorflow(const cv::Mat &frame)

* Fonksiyon girdisi görüntü matrisi referansıdır. 
* Çıktı ise `vector` tipinde OpenCV'nin `rectangular` formatındadır ve algılanan nesnelerin görüntü üzerindeki koordinat bilgilerini verir.
* `stopDetection` ile algılama işlemi ile açma-kapama işlemi gerçekleştirilir.



#### void opencv_dnn_object_detector_ros::cleanup()

* Fonksiyon çağrılırsa `stopDetection` true değeri alır.
* Eğer ki `cap` açıksa da `release` ve `cv::destroyAllWindows`  çağrılır ve kamera kapatılır.



#### void opencv_dnn_object_detector_ros::reset(bool open)

* Fonksiyon çağrıldığında ilke olarak `cleanup` fonksiyonu çağrılır. 
* Ardından `stopDetection` false olarak ayarlanır.
* Eğer ki `open` true ise tekrar açma işlemi gerçekleştirilir. 
* `initialization` fonksiyonu çağrılarak tekrardan başlatma işlemi gerçekleştirilir.

``` 	
cleanup();
stopDetection = false;
if (open)
{
	initialization();
}
```



























