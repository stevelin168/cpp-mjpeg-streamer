![C++ MJPEG over HTTP Library](doc/images/cpp_mjpeg_streamer.png)

## Features
* No OpenCV dependencies (Clear problems separation)
* Set different streams depending on HTTP GET path
* Multi-threaded streaming
* Single Header-only library

## CMake Integration
### External
```CMake
# CMakeLists.txt
find_package(nadjieb_mjpeg_streamer REQUIRED)
...
add_library(foo ...)
...
target_link_libraries(foo PRIVATE nadjieb_mjpeg_streamer::nadjieb_mjpeg_streamer)
```

### Embedded
```CMake
# CMakeLists.txt
add_subdirectory(nadjieb_mjpeg_streamer)
...
add_library(foo ...)
...
target_link_libraries(foo PRIVATE nadjieb_mjpeg_streamer::nadjieb_mjpeg_streamer)
```

## Example of Usage
### C++ Example
```c++
#include <opencv2/opencv.hpp>

#include <nadjieb/mjpeg_streamer.hpp>

// for convenience
using MJPEGStreamer = nadjieb::MJPEGStreamer;

int main()
{
    cv::VideoCapture cap(0);
    if (!cap.isOpened())
    {
        std::cerr << "VideoCapture not opened\n";
        exit(EXIT_FAILURE);
    }

    std::vector<int> params = {cv::IMWRITE_JPEG_QUALITY, 90};

    MJPEGStreamer streamer(8080);
    // By default 1 worker is used for streaming, if you want to use 4 workers
    // MJPEGStreamer streamer(8080, 4);

    streamer.start();
    while (true)
    {
        cv::Mat frame;
        cap >> frame;
        if (frame.empty())
        {
            std::cerr << "frame not grabbed\n";
            exit(EXIT_FAILURE);
        }

        // http://localhost:8080/bgr
        std::vector<uchar> buff_bgr;
        cv::imencode(".jpg", frame, buff_bgr, params);
        streamer.publish("/bgr", std::string(buff_bgr.begin(), buff_bgr.end()));

        cv::Mat hsv;
        cv::cvtColor(frame, hsv, cv::COLOR_BGR2HSV);

        // http://localhost:8080/hsv
        std::vector<uchar> buff_hsv;
        cv::imencode(".jpg", hsv, buff_hsv, params);
        streamer.publish("/hsv", std::string(buff_hsv.begin(), buff_hsv.end()));
    }
    streamer.stop();
}
```

### HTML Example
```html
<html>
  <body>
    <img src="http://localhost:8080/bgr">
    <img src="http://localhost:8080/hsv">
  </body>
</html>
```

### Compile and Run Example
Compile the example.cpp from the examples folder and run it
```sh
cd examples
mkdir build && cd build
cmake .. && make
./example
```
then open the index.html in browser to see the streams.
