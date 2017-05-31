---
layout: post
status: publish
published: true
title: CvFileStorage. How to save our custom structures with OpenCV functions
author:
  display_name: damiles
  login: admin
  email: david.millan@damiles.com
  url: ''
author_login: admin
author_email: david.millan@damiles.com
wordpress_id: 262
wordpress_url: http://blog.damiles.com/?p=262
date: '2010-02-10 16:29:54 +0100'
date_gmt: '2010-02-10 14:29:54 +0100'
---
<p><a href="{{ site.url }}/assets/2010/02/opencvGeneric.png"><img class="size-full wp-image-311" title="opencvGeneric" src="{{ site.url }}/assets/2010/02/opencvGeneric.png" alt="OpenCV" width="645" height="240" /></a></p>
In this tutorial we go to show how to save our custom structures with opencv functions.

We go to imagine we have this structure in our program.

{% highlight c++ %}
typedef struct
{
float var1;
unsigned char var2[255];
}MyStructure;
{% endhighlight %}

Opencv can save data in two formats: XML or YAML. This formats are markup languages.

For write or read a file, we go to use cvOpenFileStorage function, the functions returns  a CvFileStorage structure pointer. This pointer we use to read or write values into.

We can open a file in two ways: for read or write. The function to open file is cvOpenFileStorage

{% highlight c++ %}CvFileStorage* storage = cvOpenFileStorage( file, 0, CV_STORAGE_WRITE_TEXT );{% endhighlight %}

One time we have our CvFileStorage initialized then we can save our data. To save the data we need create Nodes. XML and YAML are markup languages, and they are defined with nodes.

First we go to create one node that contains all structure data. And inside of this node we need create one node for each variable.

For create a node we need open and close this node, as we do with xml, then we use cvStartWriteStruct and cvEndWriteStruct.

{% highlight c++ %}cvStartWriteStruct(storage, "MyStructure", CV_NODE_MAP, NULL, cvAttrList(0,0));
...
cvEndWriteStruct(storage);{% endhighlight %}

Between the start and end node we go to insert each new node with asigned variable. The variable we must save as some of this types: int, raw (array), real or string with this functions:

cvWriteInt
cvWriteRawData
cvWriteReal
cvWriteString
Then for save our structure we go to create this code:

{% highlight c++ %}
//Open file
CvFileStorage* storage = cvOpenFileStorage( file, 0, CV_STORAGE_WRITE_TEXT );
//Create our node for structure
cvStartWriteStruct(storage, "MyStructure", CV_NODE_MAP, NULL, cvAttrList(0,0));
//Write our float
cvWriteReal( storage, "var1", MyStructure->var1 );

//Write our array
cvStartWriteStruct(storage, "var2", CV_NODE_SEQ, NULL, cvAttrList(0,0));
cvWriteRawData(storage, MyStructure->var2,255,"u");
cvEndWriteStruct(storage);

//End node structure
cvEndWriteStruct(storage);

{% endhighlight %}