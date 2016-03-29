<h1 lang="en">Crosswalk Project for Android</h1>
<h1 lang="zh">安卓版Crosswalk</h1>


<p lang="en">This section describes how to create web and hybrid applications using the Crosswalk Project for the [Android operating system](http://developer.android.com/index.html).</p>
<p lang="zh">本章介绍了如何利用Crosswalk为[安卓系统](http://developer.android.com/index.html)创建web以及混合式移动应用。</p>
<p lang="en"><strong>Note：For Cordova users</strong>, follow the tutorial in [Crosswalk WebView in Cordova 4.0](/documentation/cordova.html) to easily use the advanced Crosswalk webview with your Cordova 4.0 app.</p>
<p lang="zh"><strong>注意：Cordova开发者</strong>可以利用[Crosswalk WebView in Cordova 4.0](/documentation/cordova.html)教程，轻松地让您在Cordova 4.0 app中使用高级Crosswalk webview。</p>

<p lang="en">**This tutorial explains how to:**</p>
<p lang="zh">**本章节介绍了以下及方面内容：**</p>

<p lang="en">1.  [Set up your development system *host*](/documentation/android/system_setup.html): the machine where you will be developing the application. Windows and Linux are officially supported.</p>
<p lang="zh">1.  [搭建*主机*环境](/documentation/android/system_setup.html)：主机，是您用来开发应用的机器。Crosswalk官方支持Windows系统和Linux系统。</p>

<p lang="en">2.  Set up your [Android target](/documentation/android/android_target_setup.html): the device that will run the Crosswalk application, either physical or virtual.</p>
<p lang="zh">2.  搭建[安卓平台](/documentation/android/android_target_setup.html): 安卓平台，即用来运行Crosswalk应用的物理机或者虚拟机。</p>

<p lang="en">3.  [Build a very simple HTML5 application](/documentation/android/build_an_application.html).</p>
<p lang="zh">3.  [编译一个简单的HTML5应用](/documentation/android/build_an_application.html)。</p>

<p lang="en">4.  [Run your application](/documentation/android/run_on_android.html) using the a stable release of Crosswalk.</p>
<p lang="zh">4.  [运行Crosswalk应用](/documentation/android/run_on_android.html)：通过使用一个稳定版的Crosswalk发行版本。</p>

<p lang="en">5.  [Deploy the application to the Android app store](/documentation/android/deploy_to_android_store.html).</p>
<p lang="zh">5.  [部署应用到安卓应用商店](/documentation/android/deploy_to_android_store.html)。</p>

<p lang="en">You will need to be comfortable using a command line to follow these steps. If you prefer to use a graphical integrated development environment (IDE), the free **Intel XDK** provides an alternative way to package applications for Crosswalk Android. See the [Intel XDK website](http://xdk-software.intel.com/) for more details.</p>
<p lang="zh">遵循以上步骤，您需要能够轻松地使用命令行。如果您更喜欢图形化集成环境(IDE), 免费的**Intel XDK**提供了一种打包Crosswalk安卓应用的途径。详情参见[Intel XDK官网](http://xdk-software.intel.com/)。</p>

<p lang="en">In the documentation, commands you should run in a shell are prefixed with a `>` character. On Windows, you can use the standard Windows console; on Linux, you can use a bash shell.</p>
<p lang="zh">在本篇教程中，命令行是以`>`字符开头。在Windows系统中，您可以使用标准的Windows控制台程序，在Linux系统中，您可以使用bash shell。</p>

<p lang="en">**By the end of the tutorial**, you should understand the workflow for creating Crosswalk applications from your own HTML5 projects.</p>
<p lang="zh">**阅读完本教程**，您应该了解从HTML5到创建Crosswalk应用的流程。</p>

<p lang="en">**This tutorial does not cover:**</p>
<p lang="zh">**本教程没有包含以下内容：**</p>
<ul>
<li lang="en">*   How to write HTML5 applications. We use a simple HTML5 application for this tutorial, so we can focus on the packaging and deployment aspects of Crosswalk.</li>
<li lang="zh">*   如何写HTML5应用。本教程使用了一个简单的HTML5应用作为例子，把主要篇幅放在了打包以及部署Crosswalk上面。</li>
<li lang="en">*   How to use [Crosswalk-specific APIs](/documentation/apis/web_apis.html#Experimental-APIs). The code in the tutorial should run on any modern web browser, as well as on Crosswalk.</li>
<li lang="zh">*   如何使用 [特定CrosswalkAPIs](/documentation/apis/web_apis.html#Experimental-APIs)。教程中的代码可以在任何浏览器上面，当然也能运行在Crosswalk上面。</li>
<li lang="en">*   How to compile Crosswalk itself. This is covered in the [Contribute](/contribute) section.</li>
<li lang="zh">*   如何编译Crosswalk。编译Crosswalk教程在[贡献](/contribute)这一章。</li>
</ul>
