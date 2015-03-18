---
layout: post
title:  "Tutorial on how to diploy projects to AWS Ubuntu servers"
date:   2015-03-17 21:57:22
categories: tutorial
---

### Intro

Each group will have a project dedicated Ubuntu server running somewhere on the AWS cloud. Your only connection info to that project will be the server IP address (you will recieve your server IP address and you user name by email). In this tutorial I will use the tutorial project from [previous post][previousPost] and the server with IP 52.0.4.98

[previousPost]:	http://infsci2711.github.io/tutorial/2015/01/28/tutorial-web-rest-api.html

### Logging into the server

#### From Max/Linux machine
If you have mac or you have some linux OS (e.g. Ubuntu) then you would need to use terminal to connect to the server. Simply open the terminal and type:
{% highlight xml %}
ssh YOURUSERNAME@IP
{% endhighlight %}
where YOURUSERNAME is your user name that you recieve in email and the IP is the IP address of the server.

Then enter your password (note no characters will be printed while you are entering your password) and you should be logged into the server.

#### From Windows machine
If you have Windows OS, then you need to use [PuTTy][Putty] software. Simply download it and run it. In the opened window enter server IP in the host area and leave port 22 as default. You can also save the setting for future use if you give it a name and click save button. See the picture below for the example:
![PuTTy Screen Shot](/images/puttyScreenShot.png)


[Putty]: http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html 