---
layout: post
title:  "Tutorial on how to use JerseyClientUtil to communicate to other projects via REST API from java code"
date:   2015-04-09 20:00:22
categories: tutorialPosts
---

### Intro

To communicate with other projects, you need to send REST request to their REST APIs from your JAVA code (not manually). For example when a new datasource is registered in Metastore, the Metastore project need to send requests to Presto and keyword search proejcts. When Files&APIs2DB project imports new SPSS/Dataverse file, they need to send request to the Metastore project to register new datasoruce, etc.

To send requests from your Java code, you need to use JerseyClient as I mentioned in the Piazza post with the link to the JerseyClient documentaion. To make your life easier, I have created a util class that provide doGet, doPost and doPut functions for you.

*THUS FORGET ABOUT THE JERSEYCLIENT AND USE THE JerseyClientUtil CLASS THAT I CREATED FOR YOU. YOU DON'T NEED TO ADD ANY MAVEN DEPENDNECIES OR DO ANYTHING ELSE. JUST FOLLOW THIS TUTORIAL*.

### How to use JerseyClientUtil class

First of all you need to update your MultiDBs-Utils project. If you have read previous post "Tutorial on how to configure servers with external properties file" and you did updated/rebuild MultiDBs-Utils project then you probably already have the JerseyClientUtils class. If not, then read the beginning of the post that tells you how to get the lastest code of MultiDBs-Utils project. You also need to have Tutorial project updated as well.

As I mentioned above, JerseyClientUtil class provides three methods:

+ `doGet(String restContext, String restResource)`

+ `doPost(String restContext, String restResource, Object data)`

+ `doPut(String restContext, String restResource, Object data)`

All three methods have the same frist two parameters, `String restContext` and `String restResource`.

`String restContext` is the base of the REST API URL for exapmple on my local machine it would be `localhost:7654`.

`String restResource` is the resource path. It MUST start with "/". Back to homework one, `restResource` to get the list of all persons would be `/Person`. To get a person with specific id = 1, `restResource` would be `/Person/1`, etc.

`doPost` and `doPut` also have a third parameter and is the data (request payload) that you want to send in your REST request. That can be your ViewModel classes that can be automatically converted to JSON by the Jersey.

### Example

I added an example how to use both *PropertiesManager* from the previous post and *JerseyClientUtil* in the *DemoRestApi.java* file in the *tutorialapi* project. Look at *helloWorld2* method in *DemoRestApi* class.

Assuming we have the following properties file:

{% highlight xml %}
port = 7654

# the IP and Port of the other project REST API
metastore.rest.base = http://54.152.26.131:7654
#example of the a concrete exposed resources
metastore.rest.getAllDatasources = /datasources
metastore.rest.addDatasource = /datasources/add
{% endhighlight %}

To send `GET` request to the MetaStore to get all datasources, this is how it can be done:

{% highlight java %}

// This will send GET request to 54.152.26.131:7654/datasources
Response result = JerseyClientUtil.doGet(PropertiesManager.getInstance().getStringProperty("metastore.rest.base"), PropertiesManager.getInstance().getStringProperty("metastore.rest.getAllDatasources"));

{% endhighlight %}

The result of the request will be in the *result* variable.

To send `POST` request to the MetaStore to get add new datasources, assuming the new datasource is represeted as `SomeViewModel` class, this is how it can be done:

{% highlight java %}

Response result2 = JerseyClientUtil.doPost(PropertiesManager.getInstance().getStringProperty("metastore.rest.base"), PropertiesManager.getInstance().getStringProperty("metastore.rest.addDatasource"), new SomeViewModel(123));

{% endhighlight %}

Again the result of the request will be in the *result2* variable (which you should rename of course to better name).
