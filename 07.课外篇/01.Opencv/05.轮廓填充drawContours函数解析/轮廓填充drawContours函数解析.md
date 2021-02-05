转载：https://www.cnblogs.com/long5683/p/10492180.html

函数的调用形式：

```c++
void drawContours(InputOutputArray image, InputArrayOfArrays contours, int contourIdx, const Scalar& color, int thickness=1, int lineType=8, InputArray hierarchy=noArray(), int maxLevel=INT_MAX, Point offset=Point())
```

函数参数详解：

- 第一个参数image表示目标图像，
- 第二个参数contours表示输入的轮廓组，每一组轮廓由点vector构成，
- 第三个参数contourIdx指明画第几个轮廓，如果**该参数为负值，则画全部轮廓**，
- 第四个参数color为轮廓的颜色，
- 第五个参数thickness为轮廓的线宽，如果为负值或CV_FILLED表示填充轮廓内部，
- 第六个参数lineType为线型，
- 第七个参数为轮廓结构信息，
- 第八个参数为maxLevel