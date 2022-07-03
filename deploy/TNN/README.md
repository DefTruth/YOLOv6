## Deploy YOLOv6 with TNN in C++

### Prepare third-party libs and models.
* download the pre-built libs and converted models from [Baidu Drive](https://pan.baidu.com/s/13lIk-sE3wt65DQQipSbaAQ), code: 5hfp
* or build the third_party libs from sources, [opencv](https://github.com/opencv/opencv) and [TNN](https://github.com/Tencent/TNN)

Put third_party libs into `third_party` dir and put the model files into `model` dir:  
```bash
TNN git:(main) ✗ tree . -L 1     
.
├── CMakeLists.txt      # cmake
├── README.md
├── build               # build dir
├── build-debug.sh      # build script
├── main.cpp            # demo entry
├── model               # model files (*.tnnproto&tnnmodel)
├── output.jpg          # output result
├── src                 # TNN C++ source codes
├── test.jpg            # source image
└── third_party         # third parties include and libs
```

### Check the main.cpp and CMakeLists.txt
You can write a simple main.cpp like:  
```c++
#include "yolov6.h"

static void test_yolov6()
{
  std::string proto_path = "../model/yolov6s-640x640.opt.tnnproto";
  std::string model_path = "../model/yolov6s-640x640.opt.tnnmodel";
  std::string test_img_path = "../test.jpg";
  std::string save_img_path = "../output.jpg";

  auto yolov6_det = yolov6::create(proto_path, model_path, 1); // 1 threads

  std::vector<yolov6::types::Boxf> detected_boxes;
  cv::Mat img_bgr = cv::imread(test_img_path);
  yolov6_det->detect(img_bgr, detected_boxes);

  yolov6::utils::draw_boxes_inplace(img_bgr, detected_boxes);

  cv::imwrite(save_img_path, img_bgr);

  std::cout << "Detected Boxes Num: " << detected_boxes.size() << std::endl;
}

int main(__unused int argc, __unused char *argv[])
{
  test_yolov6();
  return 0;
}
```
Future more, you can check the [CMakeLists.txt](CMakeLists.txt) for more details.

### Build YOLOv6
First, you can build the yolov6 c++ demo through (On Mac OSX and Linux)
```bash
cd ./deploy/TNN 
sh ./build-debug.sh
```
On Linux, in order to link the prebuilt libs, you need to export `third_party/lib` to LD_LIBRARY_PATH first. NOTE: you need to build [TNN](https://github.com/Tencent/TNN) from sources if you want to test this code on Linux.
```shell
export LD_LIBRARY_PATH=YOUR-PATH-TO/third_party/lib:$LD_LIBRARY_PATH
export LIBRARY_PATH=YOUR-PATH-TO/third_party/lib:$LIBRARY_PATH  # (may need)
```
Some logs may look like:
```shell
Installing TNN libs ...
-- Up-to-date: /Users/xxx/YOLOv6/deploy/TNN/build/libTNN.0.dylib
-- Up-to-date: /Users/xxx/YOLOv6/deploy/TNN/build/libTNN.dylib
Installed libs !
-- Configuring done
-- Generating done
-- Build files have been written to: /Users/xxx/YOLOv6/deploy/TNN/build
Consolidate compiler generated dependencies of target yolov6
[100%] Built target yolov6
```
Then, run the yolov6 and check the output result.
```shell
cd build
./yolov6
➜  build git:(main) ✗ ./yolov6
YOLOV6_DEBUG LogId: ../model/yolov6s-640x640.opt.tnnproto
=============== Input-Dims ==============
image_arrays: [1 3 640 640 ]
Input Data Format: NCHW
=============== Output-Dims ==============
outputs: [1 8400 85 ]
========================================
detected num_anchors: 8400
generate_bboxes num: 47
Detected Boxes Num: 5
[MEM DELETE] BasicTNNHandler done!
```

The output is:
<div align='center'>
  <img src='test.jpg' height="400px" width="350px">
  <img src='output.jpg' height="400px" width="350px">
</div>  

### How to convert YOLOv6 to other format ?
Documents about how to convert YOLOv6 to NCNN/MNN/TNN/ONNX formats, please check:  
* [知乎文章：🤓YOLOv6 ORT/MNN/TNN/NCNN C++推理部署](https://zhuanlan.zhihu.com/p/533643238)

### Reference
This code is heavily based on [lite.ai.toolkit](https://github.com/DefTruth/lite.ai.toolkit) from [DefTruth](https://github.com/DefTruth)