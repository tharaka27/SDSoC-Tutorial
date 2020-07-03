# SDSoC-Tutorial

## File Structure

* `colordetect.cpp` - C++ source file for you to edit in this tutorial. 
* `rock_landscape.jpg` - Test image for your use in testing the application. 
* `solution` folder - Source files for a completed conversion for your reference.

The source code is a very simple color detector for Blue, Green, and Orange on an input image of 1920x1080, such as the test image shown below. With the provided code and image, you will look at how to migrate the OpenCV functions and application flow using the xfOpenCV functions based on the ZCU102 reVISION platform.

1. When migrating OpenCV code to a hardware accelerated platform you should identify all the objects being used in the conversion code. Look in the `main()` and `colordetect()` functions and identify the following `cv::Mat` objects: `in_img`, `out_img`, `mask1`, `mask2`, `mask3`, `_imgrange`, and `_imghsv`.

    When running on a CPU, these objects allow for dynamic allocation and deallocation of memory as needed. However, when targeting hardware acceleration, memory requirements need to be determined at compile time. To solve this, the xfOpenCV library provides the `xf::Mat` object which facilitates memory allocation in the FPGA device. Any `cv::Mat` object used by a hardware accelerated function, whether user defined or coming from the xfOpenCV library, should be replaced with a `xf::Mat` equivalent object.  
    
    In the `main()` function, you can continue to use `cv::Mat` for input/output to the Linux system, because the code in this function is running on the CPU. However, you must convert the data from the `cv::Mat` object(s) to the `xf::Mat` object(s) for the accelerated function.

2. In the `main()` function, create input and output `xf::Mat` objects for the accelerated function, after the following if-statement:

    ```C++
    if (!in_img.data) {
	    return -1;
	}
    ```

    Add the following code:

    ```c++
    xf::Mat<XF_8UC4, HEIGHT, WIDTH, XF_NPPC1> xfIn(in_img.rows, in_img.cols);
    xf::Mat<XF_8UC1, HEIGHT, WIDTH, XF_NPPC1> xfOut(in_img.rows, in_img.cols);
    ```

   These statements create templated `xf::Mat` objects for the input and output of the hardware accelerated function. General information related to the templated object can be found in the *Xilinx OpenCV User Guide* ([UG1233](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2018_3/ug1233-xilinx-opencv-user-guide.pdf)).  

    The `xfIn` and `xfOut` objects you created have the following attributes:

    `XF_8UC4` - Defines an 8-bit unsigned-char 4-channel datatype.

    `XF_8UC1` - Defines an 8-bit unsigned-char 1-channel datatype.

    The input `xf:Mat` is declared as a 4-channel object to allow the `colorthresholding()` function to operate on the three color components (channels) of the image. The output object does not require three channels.

   `HEIGHT`/`WIDTH` - Defines the size of the input image. In the `colordetect_accel.hpp` header file, you have defined the HEIGHT and WIDTH macros for the image as **1920x1080**. Alternatively, you could statically define the maximum image size, and process variable image sizes using function parameters. 

    `XF_NPPC1` - This template parameter tells the compiler that the number of pixels processed per clock cycle is one.

