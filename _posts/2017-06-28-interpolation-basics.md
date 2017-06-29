---
layout: post
status: publish
published: true
title: 'Interpolation basic algorithms'
author:
  display_name: damiles
  login: admin
  email: david.millan@damiles.com
  url: ''
author_login: admin
author_email: david.millan@damiles.com
wordpress_id: 380
wordpress_url: http://blog.damiles.com/?p=380
date: '2017-06-28 21:59:29 +0100'
date_gmt: '2017-06-28 21:59:29 +0100'
---

It's ineresting know how works the interpolation algorithms applied to images, in this article we are going to explore basically the most common interpolation algorithms that are: Nearest, bilinear and bicubic interpolations and their formulas.

To show the artifacts of interpolations we scaled the images to show better how to affect the interpolations

Original image:
![Original image](/assets/2017/06/Lena_std.jpg)

Nearest interpolation
![Nearest interpolation](/assets/2017/06/Nearest.jpg)

Bilinear interpolation
![Bilinear interpolation](/assets/2017/06/Bilinear.jpg)

Bicubic interpolation
![Bicubic interpolation](/assets/2017/06/Bicubic.jpg)

## Nearest pixel interpolation

This is the simplest and fastest way to calculate the value of pixel. This consists in getting the nearest pixel corresponding to the original image. 

If the result of inverse function get the pixel (0.3,0) then we get the value (0,0), and if we get the pixel (0.7,0) we get the value of pixel (1,0).

This is the fastest algorithm but it makes a pixeled image.

Original:

![](/assets/2017/06/Lena_std.jpg)

Result:

![](/assets/2017/06/Nearest.jpg)

# Bilinear pixel interpolation 

This method of interpolation consists in tracing a line between the low and high rounded value of returned inverse function and calculate the resultant value with the line function.

![](/assets/2017/06/Bilinear_scale_node.gif)

The formula:
```
A’(px, py) = (1-a)(1-b)A(s, i) + a(1-b)A(s, d) +(1-a)bA(r, i) + abA(r, d) 
```

Original:

![](/assets/2017/06/Lena_std.jpg)

Result:

![](/assets/2017/06/Bilinear.jpg)

## Bicubic pixel interpolation

This method produces the better result but is the slowest algorithm. It calcules the value with a cubic curve with the 4 neighbour pixels.

![](/assets/2017/06/Bicubic_scale_node.gif)

The formula:

 ![](/assets/2017/06/Equation1.gif)

 ![](/assets/2017/06/Equation2.gif)
 
 ![](/assets/2017/06/Equation3.gif)

Original:

![](/assets/2017/06/Lena_std.jpg)

Result:

![](/assets/2017/06/Bicubic.jpg)