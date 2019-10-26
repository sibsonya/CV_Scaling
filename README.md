# Contextual image scaling
With normal scaling, the image is uniformly deformed along the entire length when resized (objects in the image are reduced along with image). The contextual image scaling algorithm takes into account the context, and the deformation occurs so that
objects retain their size. In addition, if you use a mask to select an object, then it can be removed from the image or vice versa left unchanged.

![seams](https://github.com/sibsonya/CV_Scaling/blob/master/seams.png)
# Task description
## Vertical and horizontal image compression
The idea of the algorithm is to remove the least energy seams from the image. A seam is a connected curve connecting the first and last rows / columns of an image (in the second image, the seams are highlighted in red). The energy of each point, we will call luminance gradient modulus at a given point. Image luminance - component *Y* in the color model *YUV*. The conversion from *RGB* is carried out according to the following formula: *Y = 0.299R + 0.587G + 0.114B*

If the seam passes through a small number of luminance drops, it has a small energy. Note that objects usually have a more complex structure than the background, which means that seams, which will pass through them, will have a higher energy, therefore we will not delete them. We need to implement a function that would reduce the size of the image by 1 pixel in a given direction.
### The seam removal algorithm (for the case of horizontal reduction):
1. We consider the image energy (we find the norm of the gradient at each point). To find gradient, it is necessary to find the partial derivatives in each of the directions (in X and in Y). In order to find the derivative with respect to X, you need for each pixel from the right neighbor subtract the left (we get the difference derivative of the second-order derivative of accuracy): 

   *I'(x, y) = I(x + 1, y) − I(x − 1, y)* 

   The derivatives at the image boundary are calculated using a first-order approximation:

   *I'(0, y) = I(1, y) − I(0, y)*

   *I'(w − 1, y) = I(w − 1, y) − I(w − 2, y)*

   Here *w* is the image width, indexing starts from zero. 

   Similarly, you can find partial derivative with respect to Y (subtract the upper pixel from the lower one). Now in order to
find the norm of the gradient, it is necessary for each pixel to extract the root of the sum of squares partial derivatives.

![borders](https://github.com/sibsonya/CV_Scaling/blob/master/borders.png)

2. Finding a seam with minimum energy:
   - Create a matrix of the same size as the original image, and initialize the first row of this matrix with the energy of the corresponding points.
   - Fill in our matrix line by line. For each point, we find the minimum of the three upper neighbors (the side pixels have two neighbors; if the neighbors have the same values, then when we form the seam, we take the left one), add its value and the energy value of this point, write the result in the matrix.
   

      | 3 | 4 | 3 | 5 |                            
      |---|---|---|---|                            
      | **5** | **4** | **5** | **6** |           
      | **1** | **1** | **1** | **1** |
      
      
      | 3 | 4 | 3 | 5 |                            
      |---|---|---|---|                            
      | **8** | **7** | **8** | **9** |           
      | **8** | **8** | **8** | **9** |
      
      In this example: in the first matrix - energy values; in the second, the matrix, required to build.
   - Now we select the minimum value in the last line (if there are several, then we take the leftmost one). This point belongs to the minimum seam. Now we can trace all points that belong to the minimum seam.
