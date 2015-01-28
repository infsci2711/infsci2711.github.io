---
layout: post
title:  "Tutorial on simple Web application with RESTful API on JAVA"
date:   2015-01-28 10:57:22
categories: tutorial
---

### Intro

This is the tutorial that should have taken place in class on January 27th, 2015, but was not successful due to the problems with students computers.

The **goal** is to build a simple web application which has a pure HTML/Javascript web client (not hosted) and RESTful API written in JAVA with the help of [Jersey REST Framework][Jersey] and [Jetty Web Server][Jetty]. Java project will be managed by [Maven][Maven].

### Step 1: Get the source code

Each team has 4 four repositories:
![GitHub Team Repos](/images/gitHubTeamRepos.png)

Two of those repositories are specific (e.g. `infsci2711\MultiDBs-FilesAPIs2DBs-Server` and `infsci2711\MultiDBs-FilesAPIs2DBs-WebClient` for the *MultiDBs-FilesAPIs2DBs* team) for each team and the other two (`infsci2711\MultiDBs-Utils` and `infsci2711\infsci2711.github.io`) are shared between all teams.

The purpose of each respository ("_FilesAPIs2DBs_" part of them name will be different for each team):

+ `infsci2711\MultiDBs-FilesAPIs2DBs-Server` - repository where you put your Java code with RESTful APIs and business logic

+ `infsci2711\MultiDBs-FilesAPIs2DBs-WebClient` - repostiry where you put your web client code (html/javascript/css/etc)

+ `infsci2711\MultiDBs-Utils` - repository shared between all teams which contains all utility classes that might be useful for more than one team

+ `infsci2711\infsci2711.github.io` - repository with the source code of the class website


Each member need to clone the first three repositories (the website repo is optional and might be needed if you will need to contribute to the class website). You will use your project specific repositories later when you work on your project, for this demo only need the `infsci2711\MultiDBs-Utils` repository.

Also clone `infsci2711\tutorial` ([https://github.com/infsci2711/tutorial](https://github.com/infsci2711/tutorial)) and `infsci2711\tutorial-WebClient` ([https://github.com/infsci2711/tutorial-WebClient](https://github.com/infsci2711/tutorial-WebClient)) repositories that will be used for this tutorial. Tutorial repositories resembel exactly the same structure as your team specific repositories, so you can play/mess around with tutorial before doing the actual work in your teams repos.

### Step 2: Maven (apparently it is optional)

Java projects are managed by [Maven][Maven] and thus to build projects you need to use Maven. If you want to use Maven from command prompt (terminal), then follow installation instuctions on the Maven web site. However, as I learned it yesterday, you don't need to install Maven separately if you only going to use it via an IDE (you need an IDE with maven though).

### Step 3: IDE (with maven plugin)

You can use any IDE you want. I am going to show all examples with [Eclipse][Eclipse]. In any case, I believe, you IDE need to "understand" Maven projects. New version of [Eclipse][Eclipse] (after Kepler) and [NetBeans](https://netbeans.org/) have Maven plugin by default (I don't know about the [IntelliJ IDEA](https://www.jetbrains.com/idea/), but I am sure the situation is the same). If you have older version of Eclipse, for example, you can install [m2eclipse](http://eclipse.org/m2e/) plugin separately.

### Step 4: Import projects to IDE

Import *MultiDBs-Utils* and *tutorial* projects to IDE. Here is how you can do that in Eclipse:

1. Open Eclipse

2. Select File->Import...

3. In the Import dialog type Maven in "type filet text" text box to filter all import source options and then select "Existing Maven Projects" and click Next
![Eclipse Import Dialog](/images/importDialog.png)

4. Click the Browse... button and select project folder. 

   For *MultiDBs-Utils* project the import dialog should look like this:

   ![Eclipse Import Dialog](/images/importDialogUtilsProject.png)

   For *tutorial* project the import dialog should look like this (uncheck the root pom file):

   ![Eclipse Import Dialog](/images/importDialogTutorialProject.png)

5. Click Next and then Finish. You should see three projects in the Project Explorer (you might be missing *target* folders, but don't worry about those):
![Three projects in the project explorer](/images/eclipseProjectExplorerThreeProjects.png)

Each project has the same folder structure. Here is what each folder is used for:

* *src/main/java* - is the place where all packages and **business logic** code will go

* *src/main/resources* - is the place where resource files (e.g. logo, properties, etc.) will go

* *src/test/java* - is the place where all packages and **test** code will go

* *src/main/java* - is the place where resource files (e.g. logo, properties, etc.) that are needed for the test will go

The *tutorial* maven project actually consists of two maven modules (*tutorialserver* and *tutorialserverapi*) that now looks like just other normal java project:

* *tutorialserver* project should be used for the business logic of your project without including any REST related stuff. Think of this a just a library that will be used in some other projects.

* *tutorialserverapi* is the project that starts a web server and exposes RESTful API. No actual business logic that solve your team's tasks should be in this. This project will depend on the *tutorialserver*.

### Step 5: Maven Install MultiDBs-Utils proejct

We are going to use some utility classes from *MultiDBs_Utils* project in our tutorial project. Since we just cloned the Utils project, it is not available in our local maven repository. There are two options to install that project:

* Using maven via command prompt (terminal):

   * open command prompt (terminal)

   * cd to the location of the *MultiDBs_Utils* project

   * type `mvn install`

*OR*

* Using IDE:
   
   * Right click on the *MultiDBs-Utils* project in Eclipse
   * Select Run As -> Maven Install

If everything worked correctly you should see a message in terminal or in console output similar to this one:
    
{% highlight xml %}
[INFO] -----------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------
[INFO] Total time: 2.087s
[INFO] Finished at: Wed Jan 28 13:24:40 CST 2015
[INFO] Final Memory: 8M/19M
[INFO] ------------------------------------------
{% endhighlight %}

### Step 6: Add *MultiDBs-Utils* and *tutorialserver* as maven dependecny to *tutorialserverapi* project

* Open *pom.xml* file in the *rutorialseverapi
* Find `<dependencies>` tag
* Add following two dependencies:

{% highlight xml %}
<dependency>
	<groupId>edu.pitt.sis.infsci2711</groupId>
	<artifactId>multidbsutils</artifactId>
	<version>0.1-SNAPSHOT</version>
</dependency>

<dependency>
	<groupId>edu.pitt.sis.infsci2711</groupId>
	<artifactId>tutorialserver</artifactId>
	<version>0.1-SNAPSHOT</version>
</dependency>
{% endhighlight %}

The first dependency is to the *MultiDBs-Utils* project since we need to use some classes from that project and the second dependency is to the *tutorialserver* project since that project will do all the work and the *tutorialserverapi* project will "forward" all request to it.

### Step 7: Demo RESTful API call

Let's work on the *tutorialserverapi* project first. 

Right click on the *src/main/java* in the Project Explorer and select *New -> Package* to create a new package. In the *New Java Package* dialog enter the name of the new package as *edu.pitt.sis.infsci2711.tutorial*. 

---

**NOTE**: all packages in your projects should have *edu.pitt.sis.infsci2711.multidbs.* prefix followed by your project name. E.g., all packages in the *MultiDBs-FilesAPIs2DBs* project should have this beginning: *edu.pitt.sis.infsci2711.multidbs.filesapis2dbs*

---
<br/>
After you added the package, right click on it in the *Project Explorer* and select *New -> Class*. In the *New Java Class* dialog enter name for the new class as *TutorialServer*.

Do similar steps to add *edu.pitt.sis.infsci2711.tutorial.rest* package and *DemoRestApi* class in that package.

You should have project structure like this:

![API project with 2 packages and 2 classes](/images/apiprojectWith2Pack2Classes.png)

#### Step 7.1: Starting Jetty Server

Edit *TutorialServer.java* file to make it like this:

{% highlight java %}
package edu.pitt.sis.infsci2711.tutorial;

import edu.pitt.sis.infsci2711.multidbs.utils.JerseyJettyServer;

public class TutorialServer {
	
	public static void main(final String[] args) throws Exception {
		JerseyJettyServer server = new JerseyJettyServer(7654, "edu.pitt.sis.infsci2711.tutorial.rest");
		server.start();
	}
}
{% endhighlight %}

[Jersey]:	https://jersey.java.net/
[Jetty]:	http://eclipse.org/jetty/
[Maven]:	http://maven.apache.org/
[Eclipse]:	https://eclipse.org/
