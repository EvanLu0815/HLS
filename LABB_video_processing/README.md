# Video System

## Brief Description of Function

###### System

System includes:

 - Main function: function calls.
   	- PixelIni: Generate pixel_in to be operated.
   	- video_filter_rescale: Rescale pixel_in according to the format of video.
   	- PrintPixel: Print the results of each function.
 - Kernel
   	- filter: Do the multiply accumulate(MAC) on 3$\times$3 window and 3$\times$3 h, which is element-wise multiplication not matrix multiplication.
   	- video_2dfilter: prepare window for filter. In this lab, how to prepare the window is the main task.

###### video_2dfilter

- Simply access 3$\times$3 pixels from pixel_in and store to window.

###### video_2dfilter_linebuffer

- In each iteration, only access one pixel from pixel_in.
- Use line buffer to store pixel_in.
- Both line buffer and window do the spatial shift to achieve data reuse.

###### video_2dfilter_linbuffer_extended

- Base on the previous function, prepare the first pixel_in and window in the first iteration and write the pixel_out in the second iteration and so on and so forth.

###### video_2dfilter_linebuffer_extended_constant

- Fix the problem of window accessing those pixel beyond boundary. 

## Build Flow

1. Open the Vitis HLS 2021.2
2. Include the ```video_top.cpp```, ```video_filter_rescale.cpp```, and ```video_common.h``` to test bench
3. Include the ```video_2dfilter.cpp``` and ```video_common.h``` to src.
4. Run C Synthesis.
5. Change ```video_2dfilter.cpp``` to ```video_2dfilter_linebuffer.cpp```, ```video_2dfilter_linebuffer_extended.cpp```, or ```video_2dfilter_linebuffer_extended_constant.cpp```  and run C synthesis.
6. Adjust those ```#pragma``` and analyze the synthesis report. 

## Result/ Analysis

1. We cannot achieve II = 1 when we access  3$\times$3 pixels from pixel_in for each time we call filter.

2. We can achieve II = 1 when we access only one pixel from pixel_in for each time we call filter.

3. ```rewind``` will help to decrease the latency that main function prepares to access the next input.

4. ```enable_flush``` will not help to decrease the loop latency in some cases.

   When flush the pipeline, the pipeline will shut down all the successive stage until the final input has been processed.

   $\to$ If the time of pipeline stalling is shorter than or equals to that of completing all the older inputs and resume pipeline, ```enable_flush``` is not a good choice.

5. In some cases, ```rewind``` and ```enable_flush``` are useless. They will cause timing violation.