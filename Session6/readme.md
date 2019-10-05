**Objective**
___

To use Advanced Convolutions

* Run the Base Model.After training the network, whatever accuracy you get is your base accuracy. Epochs = 100
* Fix the network above:
  1. remove dense
  2. add layers required to reach RF
  3. fix kernel scaleup and down (1x1)
  4. see if all dropouts are properly placed
  5. Get accuracy more than the base accuracy in less number 100 epochs. Hint, you might want to use "border_mode='same',"
---
* Rewrite it again using these convolutions in the order given below:
  1. Normal Convolution
  2. Spatially Separable Convolution  (Conv2d(x, (3,1)) followed by Conv2D(x,(3,1))
  3. Depthwise Separable Convolution
  4. Grouped Convolution (use 3x3, 5x5 only)
  5. Grouped Convolution (use 3x3 only, one with dilation = 1, and another with dilation = 2) 
  6. You must use all of the 5 above at least once
  7. Train this new model for 50 epochs. 
