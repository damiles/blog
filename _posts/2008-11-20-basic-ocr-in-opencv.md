---
layout: post
status: publish
published: true
title: Basic OCR in OpenCV
author:
  display_name: damiles
  login: admin
  email: david.millan@damiles.com
  url: ''
author_login: admin
author_email: david.millan@damiles.com
wordpress_id: 93
wordpress_url: http://blog.damiles.com/?p=93
date: '2008-11-20 21:59:29 +0100'
date_gmt: '2008-11-20 19:59:29 +0100'
---
In this tutorial we go to create a basic number OCR. It consist to classify a handwrite number into his class.
To do it, we go to use all we learn in before tutorials, we go to use a simple <a title="Basic Painter" href="/2008/11/07/basic-painter-in-opencv.html" target="_blank">basic painter</a> and <a title="pattern recognition" href="/2008/11/14/the-basic-patter-recognition-and-classification-with-opencv.html" target="_blank">the basic pattern recognition and classification with openCV</a> tutorial.

<a class="download" title="Demo Source Basic OCR" href="https://github.com/damiles/basicOCR" target="_blank">Demo Source from GitHub</a>

In a typical pattern recognition classifier consist in three modules:

<a href="{{ site.url }}/assets/2008/11/pr.gif"><img class="aligncenter size-full wp-image-94" title="Pattern recognition modules" src="{{ site.url }}/assets/2008/11/pr.gif" alt="" width="400" height="600" /></a>


<!--more-->Preprocessing: in this module we go to process our input image, for example size normalize, convert color to BN...

Feature extraction: in this module we convert our image processed to a characteristic vector of features to classify, it can be the pixels matrix convert to vector or get contour chain codes data representation

Classification module get the feature vectors and train our system or classify an input feature vector with a classify method as knn.

In this basic OCR we go to use this graph:

<a href="{{ site.url }}/assets/2008/11/knnc-graph.jpg"><img class="aligncenter size-full wp-image-95" title="knnc-graph" src="{{ site.url }}/assets/2008/11/knnc-graph.jpg" alt="" width="400" height="500" /></a>

Where we get a train set and test set of image to train and test our classifier method (knn)

We have a 1000 handwrite images, 100 images of each number. We get 50 images of each number (class) to train and other 50 to test our system.

<a href="{{ site.url }}/assets/2008/11/numbers.gif"><img class="aligncenter size-full wp-image-97" title="numbers" src="{{ site.url }}/assets/2008/11/numbers.gif" alt="" width="400" height="400" /></a>

Then the first work we do is pre-process all train image, to do it we create a preprocessing function. In this function we get a image and a new width and height we want as result of preprocessing, then the function return a normalized size with bounding box image. You can see more clear the process in this graph:

<a href="{{ site.url }}/assets/2008/11/preprocesing.gif"><img class="aligncenter size-full wp-image-100" title="preprocesing" src="{{ site.url }}/assets/2008/11/preprocesing.gif" alt="" width="400" height="600" /></a>

Pre-processing code:

{% highlight c++ %}
void findX(IplImage * imgSrc, int * min, int * max) {
  int i;
  int minFound = 0;
  CvMat data;
  CvScalar maxVal = cvRealScalar(imgSrc - > width * 255);
  CvScalar val = cvRealScalar(0);
  //For each col sum, if sum < width*255 then we find the min
  //then continue to end to search the max, if sum< width*255 then is new max
  for (i = 0; i < imgSrc - > width; i++) {
    cvGetCol(imgSrc, & data, i);
    val = cvSum( & data);
    if (val.val[0] < maxVal.val[0]) { * max = i;
      if (!minFound) { * min = i;
        minFound = 1;
      }
    }
  }
}

void findY(IplImage * imgSrc, int * min, int * max) {
  int i;
  int minFound = 0;
  CvMat data;
  CvScalar maxVal = cvRealScalar(imgSrc - > width * 255);
  CvScalar val = cvRealScalar(0);
  //For each col sum, if sum < width*255 then we find the min
  //then continue to end to search the max, if sum< width*255 then is new max
  for (i = 0; i < imgSrc - > height; i++) {
    cvGetRow(imgSrc, & data, i);
    val = cvSum( & data);
    if (val.val[0] < maxVal.val[0]) { * max = i;
      if (!minFound) { * min = i;
        minFound = 1;
      }
    }
  }
}
CvRect findBB(IplImage * imgSrc) {
  CvRect aux;
  int xmin, xmax, ymin, ymax;
  xmin = xmax = ymin = ymax = 0;

  findX(imgSrc, & xmin, & xmax);
  findY(imgSrc, & ymin, & ymax);

  aux = cvRect(xmin, ymin, xmax - xmin, ymax - ymin);

  //printf("BB: %d,%d - %d,%d\n", aux.x, aux.y, aux.width, aux.height);

  return aux;

}

IplImage preprocessing(IplImage * imgSrc, int new_width, int new_height) {
  IplImage * result;
  IplImage * scaledResult;

  CvMat data;
  CvMat dataA;
  CvRect bb; //bounding box
  CvRect bba; //boundinb box maintain aspect ratio

  //Find bounding box
  bb = findBB(imgSrc);

  //Get bounding box data and no with aspect ratio, the x and y can be corrupted
  cvGetSubRect(imgSrc, & data, cvRect(bb.x, bb.y, bb.width, bb.height));
  //Create image with this data with width and height with aspect ratio 1
  //then we get highest size betwen width and height of our bounding box
  int size = (bb.width > bb.height) ? bb.width : bb.height;
  result = cvCreateImage(cvSize(size, size), 8, 1);
  cvSet(result, CV_RGB(255, 255, 255), NULL);
  //Copy de data in center of image
  int x = (int) floor((float)(size - bb.width) / 2.0 f);
  int y = (int) floor((float)(size - bb.height) / 2.0 f);
  cvGetSubRect(result, & dataA, cvRect(x, y, bb.width, bb.height));
  cvCopy( & data, & dataA, NULL);
  //Scale result
  scaledResult = cvCreateImage(cvSize(new_width, new_height), 8, 1);
  cvResize(result, scaledResult, CV_INTER_NN);

  //Return processed data
  return *scaledResult;

}
{% endhighlight %}

We use the function getData of basicOCR class to create the train data and train classes, this function get all images under OCR folder to create this train data, the OCR forlder is structured with 1 folder to each class and each file have are pbm files with this name cnn.pbm where c is the class {0..9} and nn is the number of image {00..99}

Each image we get is pre-processed and then convert the data in a feature vector we use.

basicOCR.cpp getData code:

{% highlight c++ %}
void basicOCR::getData()
{
IplImage* src_image;
IplImage prs_image;
CvMat row,data;
char file[255];
int i,j;
for(i =0; i&lt;classes; i++){
for( j = 0; j&lt; train_samples; j++){

//Load file
if(j&lt;10)
sprintf(file,&quot;%s%d/%d0%d.pbm&quot;,file_path, i, i , j);
else
sprintf(file,&quot;%s%d/%d%d.pbm&quot;,file_path, i, i , j);
src_image = cvLoadImage(file,0);
if(!src_image){
printf(&quot;Error: Cant load image %s\n&quot;, file);
//exit(-1);
}
//process file
prs_image = preprocessing(src_image, size, size);

//Set class label
cvGetRow(trainClasses, &amp;row, i*train_samples + j);
cvSet(&amp;row, cvRealScalar(i));
//Set data
cvGetRow(trainData, &amp;row, i*train_samples + j);

IplImage* img = cvCreateImage( cvSize( size, size ), IPL_DEPTH_32F, 1 );
//convert 8 bits image to 32 float image
cvConvertScale(&amp;prs_image, img, 0.0039215, 0);

cvGetSubRect(img, &amp;data, cvRect(0,0, size,size));

CvMat row_header, *row1;
//convert data matrix sizexsize to vecor
row1 = cvReshape( &amp;data, &amp;row_header, 0, 1 );
cvCopy(row1, &amp;row, NULL);
}
}
}{% endhighlight %}

After processed and get train data and classes whe then train our model with this data, in our sample we use knn method then:

{% highlight c++ %}knn=new CvKNearest( trainData, trainClasses, 0, false, K );{% endhighlight %}

Then we now can test our model, and we can use the test result to compare to another methods we can use, or if we reduce the image scale or similar. There are a function to create the test in our basicOCR class, test function.

This function get the other 500 samples and classify this in our selected method and check the obtained result.

{% highlight c++ %}void basicOCR::test(){
IplImage* src_image;
IplImage prs_image;
CvMat row,data;
char file[255];
int i,j;
int error=0;
int testCount=0;
for(i =0; i&lt;classes; i++){
for( j = 50; j&lt; 50+train_samples; j++){

sprintf(file,&quot;%s%d/%d%d.pbm&quot;,file_path, i, i , j);
src_image = cvLoadImage(file,0);
if(!src_image){
printf(&quot;Error: Cant load image %s\n&quot;, file);
//exit(-1);
}
//process file
prs_image = preprocessing(src_image, size, size);
float r=classify(&amp;prs_image,0);
if((int)r!=i)
error++;

testCount++;
}
}
float totalerror=100*(float)error/(float)testCount;
printf(&quot;System Error: %.2f%%\n&quot;, totalerror);

}{% endhighlight %}

Test use the classify function that get image to classify, process image, get feature vector and classify it with a find_nearest of knn class. This function we use to classify the input user images:

{% highlight c++ %}float basicOCR::classify(IplImage* img, int showResult)
{
IplImage prs_image;
CvMat data;
CvMat* nearest=cvCreateMat(1,K,CV_32FC1);
float result;
//process file
prs_image = preprocessing(img, size, size);

//Set data
IplImage* img32 = cvCreateImage( cvSize( size, size ), IPL_DEPTH_32F, 1 );
cvConvertScale(&amp;prs_image, img32, 0.0039215, 0);
cvGetSubRect(img32, &amp;data, cvRect(0,0, size,size));
CvMat row_header, *row1;
row1 = cvReshape( &amp;data, &amp;row_header, 0, 1 );

result=knn-&gt;find_nearest(row1,K,0,0,nearest,0);

int accuracy=0;
for(int i=0;i&lt;K;i++){
if( nearest-&gt;data.fl[i] == result)
accuracy++;
}
float pre=100*((float)accuracy/(float)K);
if(showResult==1){
printf(&quot;|\t%.0f\t| \t%.2f%%  \t| \t%d of %d \t| \n&quot;,result,pre,accuracy,K);
printf(&quot; ---------------------------------------------------------------\n&quot;);
}

return result;

}{% endhighlight %}

All work or training and test is in basicOCR class, when we create a basicOCR instance then only we need call to classify function to classify our input image. Then we go to use basic Painter we create before in other tutorial to user interactivity to draw a image and classify it.

<a class="download" title="Demo Source Basic OCR" href="https://github.com/damiles/basicOCR" target="_blank">Demo Source</a>
