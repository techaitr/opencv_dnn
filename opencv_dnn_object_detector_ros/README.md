# opencv_dnn_object_detector_ros

ROS Melodic + OpenCV DNN ile kamera üzerinden insan/nesne algılama paketi.

Desteklenen modeller:
- **darknet** — YOLO (ör. yolov4-tiny)
- **tensorflow** — SSD MobileNet

Kurulum, parametreler ve çalıştırma için repo kökündeki [README.md](../README.md) dosyasına bakın.

Hızlı çalıştırma:

```bash
roslaunch opencv_dnn_object_detector_ros opencv_dnn_object_detector_ros.launch
```

Model yollarını `config/opencv_dnn_object_detector_ros.yaml` içinde kendi makinenize göre güncelleyin.
