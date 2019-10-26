# Contextual image scaling
With normal scaling, the image is uniformly deformed along the entire length when resized (objects in the image are reduced along with image). The contextual image scaling algorithm takes into account the context, and the deformation occurs so that
objects retain their size. In addition, if you use a mask to select an object, then it can be removed from the image or vice versa left unchanged.

![seams](https://github.com/sibsonya/CV_Scaling/blob/master/imgs/seams.png)
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

![borders](https://github.com/sibsonya/CV_Scaling/blob/master/imgs/borders.png)

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
3. Remove this seam from the image.

## Working with mask + image extension
The description below will be given for the case of horizontal image changes (for the vertical everything is similar).
Using the mask, we can control the formation of seams. We can either increase energy of the corresponding points (then these points will remain on the final image), or vice versa reduce (then these points will be removed from the image). We will increase or decrease the mask by a known large amount: the product of the number of image lines and the number of image columns by 256 (the upper estimate of the gradient value at a point). Using this method, we can, for example, highlight a person’s face with a mask, and it will not be deformed as a result of our algorithm. It is necessary to realize the ability to remove objects that are highlighted by the mask (reduce the energy of the corresponding points), and protect them from deformation (increase the energy of the corresponding points)

![mask](https://github.com/sibsonya/CV_Scaling/blob/master/imgs/mask.png)

To enlarge the image, you need to find the minimum seam and insert a new one to the right of it, which would be an averaging of the minimum seam and the next one (for each pixel of the minimum seam we take its raight neighbor). Note that if we carry out several iterations of our algorithm, then the minimum seam will be the same each time. To avoid this, it is necessary to increase the value of the mask at the points of this seam (this is the way we use the mask in case of expansion).

# Program interface, data and script for testing
Агтсешщт **seam_carve** with the following arguments:
1. The input image.
2. The operation mode of the algorithm, one of four lines:
   ’horizontal shrink’
   ’vertical shrink’
   ’horizontal expand’
   ’vertical expand’
3. (Optional argument). Image mask - a single-channel image that matches the size of the input image. The mask consists of elements {-1, 0, +1}. -1 means pixels to be deleted, +1 means saving. 0 means that the pixel energy does not need to be changed.

The function returns a triple - the changed image and mask, as well as the seam mask (1 - pixels that belong to the seam; 0 - pixels that do not belong to the seam). In the mask, you must also remove or add the corresponding seam.

Testing data - images and masks in png format. The answer for each pair of images is seam coordinates serialized using the pickle module for all possible operating modes. For all intermediate calculations (calculation of gradients, energy), type float64 is used.

The operation of the algorithm can be visualized using the gui.py script (it can help you with debugging). It has an optional argument - the path to the image. The script requires PyQt4. Try to run your algorithm for different versions of images and masks. There are 8 stitches in the layout for each test image.

# Useful resources
[Algorithm video demonstration](https://www.youtube.com/watch?v=vIFCV2spKtg)

[Seam carving for content-aware image resizing](http://citeseerx.ist.psu.edu/viewdoc/downloaddoi=10.1.1.570.6321&rep=rep1&type=pdf) - original article with a lot of explanatory examples, recommended for a detailed acquaintance with the algorithm

[Wikipedia article on seam carving](https://en.wikipedia.org/wiki/Seam_carving)

