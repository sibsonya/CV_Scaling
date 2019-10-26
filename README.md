# Contextual image scaling
With normal scaling, the image is uniformly deformed along the entire length when resized (objects in the image are reduced along with image). The contextual image scaling algorithm takes into account the context, and the deformation occurs so that
objects retain their size. In addition, if you use a mask to select an object, then it can be removed from the image or vice versa left unchanged.

![seams](https://github.com/sibsonya/CV_Scaling/blob/master/seams.png)
# Task description
## Vertical and horizontal image compression
The idea of the algorithm is to remove the least energy seams from the image. A seam is a connected curve connecting the first and last rows / columns of an image (in the second image, the seams are highlighted in red). The energy of each point, we will call luminance gradient modulus at a given point. Image brightness - component Y in the color model *YUV*. The conversion from *RGB* is carried out according to the following formula:
                                            *Y = 0.299R + 0.587G + 0.114B*

