---
layout: post
title:  "Tutorial on how to diploy projects to AWS Ubuntu servers"
date:   2015-03-17 21:57:22
categories: tutorialPosts
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

Then click open and in the opened window enter your username, hit enter, then enter your password.

[Putty]: http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html 

### Preparing Ubuntu

When you log in first time, the server has "fresh" installation of Ubuntu OS, meaning that no additional software (such as Maven, Git, etc.) is installed. Everyone in one team share the same computer, so all the software that you need for your project should only be installed once (you need to talk to each other and organize this activity).

Here are several example of what you might need to install (you can simply copy and past line by line to the terminal):

{% highlight xml %}
sudo apt-get install maven
sudo apt-get install git
sudo apt-get install openjdk-7-jdk
sudo apt-get install mysql-server
sudo apt-get install nginx
{% endhighlight %}

As I menioned in the class, each project would need to create a *startup.sh* scrip file that should bring the "fresh" OS installation to the state to be able to run your project without any other manual work. So I suggest you to start writing the script as soon as possible, so you will not forget anything. 

Also the *startup.sh* should have any other required operations scripted. E.g. creation of the database and tables if your project needs any. You can look at my example [here](https://github.com/infsci2711/tutorial/blob/master/setup.sh).

### Getting the source code to the server

*You should never copy any source code or any script file manually to the server. Everything important should go via github!* 

To get your code from github follow the following steps:

1. Create project folder under the /opt folder:

{% highlight xml %}
cd /opt
sudo mkdir project
cd project
{% endhighlight %}

2. Clone all required github repos. For example, to clone the tutorial project following command will need to be exectued:

{% highlight xml %}
sudo git clone https://github.com/infsci2711/tutorial.git
{% endhighlight %}

You can get your project git HTTPS URL on the git hub web site.

The server code will be served directly from the *project* folder with Jetty web server, however to server "static" Web UI (Client code) you have several options:
a. To serve it with the *nginx* (this will be used in this tutorial)
b. To serve it together with the server code with Jetty (you need to figure out by yourself how to do that if you want to)
c. To server with any other web servers (e.g. Apache, tomcat, etc. (you need to figure out by yourself how to do that if you want to))

You can clone the Client code into the *project* directory as well however eventually all client code (e.g. html, js, css files) need to be under nginx root web folder which is */usr/share/nginx/html*). There are several ways to do that:
a. Create symlink to the client code in the project folder
b. Copy client code from the project folder to the nginx folder
c. git clone your client code directly in the nginx folder

Here is the example how to make it work with the symlink:

{% highlight xml %}
cd /usr/share/nginx
sudo rm -R html
sudo ln -sv /opt/project/tutorial-WebClient html
{% endhighlight %}

Note for the above code to work, nginx need to be configured to follow symlink (for it was working out of the box, let me know if it doesn't work for you). Also in the above commands */opt/project/tutorial-WebClient* points to the client code that I cloned from github. You would need to replace it for your project. Also, you might need to make *project* folder searchable (e.g. give x permission with "chmod 755 project" command)

Even though we have not talked about how to start the server, by now you should be able to check if UI "works" (at least shows correct page without back end functionality). Simply type your server IP address in the browser. You should see your UI.

The only change that I needed to make to the UI code is in the *personViewModel.js* file (changed the restBaseUrl to point to the server IP):
{% highlight javascript %}
var restBaseUrl = "http://52.0.4.98:7654/";
{% endhighlight %}

Here is the screen shot of the projects folder that I have on the server for this example:

![Project Folder on AWS](/images/projectFolderonAWS.png)

### Building and Starting your server code

There is one thing that you need to add to your severapi project (in my case it was tutorialapi project) pom file. One person from each team needs to add the following to the project section of the pom file:

{% highlight xml %}
<build>
	<plugins>
				
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-jar-plugin</artifactId>
			<version>2.2</version>
			<!-- nothing here -->
		</plugin>
		
		<plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>2.1</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                    <configuration>
                        <transformers>
                        <!--  use transformer to handle merge of META-INF/services - see http://java.net/jira/browse/JERSEY-440?focusedCommentId=14822&page=com.atlassian.jira.plugin.system.issuetabpanels%3Acomment-tabpanel#action_14822 -->
                            <transformer
                                implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer" />
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
			                	<mainClass>edu.pitt.sis.infsci2711.tutorial.TutorialServer</mainClass>
			                </transformer>
                        </transformers> 
                        <filters>
                            <!--  filter to address "Invalid signature file" issue - see http://stackoverflow.com/a/6743609/589215-->
                            <filter>
                                <artifact>*:*</artifact>
                                <excludes>
                                    <exclude>META-INF/*.SF</exclude>
                                    <exclude>META-INF/*.DSA</exclude>
                                    <exclude>META-INF/*.RSA</exclude>
                                </excludes>
                            </filter>
                        </filters>
                    </configuration>
                </execution>
            </executions>
        </plugin>

	</plugins>
</build>
{% endhighlight %}

Don't forget to replace *<mainClass>edu.pitt.sis.infsci2711.tutorial.TutorialServer</mainClass>* with your main class.
Note: do that on the local machine and then commit to github. To get your latest commit to the server, simply use *git pull* command.

Once you get all the required code, you need to build it first. Start with those projects that are independent. For example, the I would start with *MultiDBs-Utils* since it doesn't depend on any other projects. To build the project, run mvn instal like this:

{% highlight xml %}
cd MultiDBs-Utils
mvn install
{% endhighlight %}

Do the same for all projects you have.

Once you run the *mvn install* command on the sever project, go to the *serverapi/target* folder. There should be two jar files. If you see only one jar file, then something went wrong.

If you do see two jar file, then run the following command to start your server:

{% highlight xml %}
nohup java -jar tutorialserverapi-0.1-SNAPSHOT.jar > log.out 2> error.log < /dev/null &
{% endhighlight %}

Replace *tutorialserverapi-0.1-SNAPSHOT.jar* with your jar name. To test if the server is running, try to run:
{% highlight xml %}
lsof -i:7654
{% endhighlight %}

Make sure in your code the server is listening to the port 7654, since only this port and port 80 are opented to the outside world.

If you think that everything is working, try to test it in your browser. For the tutorial project, I would simply type sever IP address in the browser.

### Getting code updates after new commits

After you do more commits to github with new functionality, you would need to "redploy" your project. 

Connect to the server, stop your sever project (Jetty server) by running following command:

Then go to each repository folder *git pull*.

Start your server project (Jetty server).
P.S.: You might need to restart any other servers/services if you use any.