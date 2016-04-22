# 主机和目标机配置

参考[系统设置](/documentation/android/system_setup.html)设置Crosswalk开发环境。

您也需要设置[设置安卓目标机](/documentation/android/android_target_setup.html)。作为测试，您可以使用[自己的安卓设备](/documentation/android/android_target_setup.html#Android-Device)或者[虚拟设备](/documentation/android/android_target_setup.html#android-emulator)。教程使用的是：

* Android 4.2.2的x86版ZTE Geek手机
* Android 4.0.4的ARM版HTC oneX

<h2 id="Project-outline">工程概览</h2>

方便起见，插件和应用位于同一个顶层目录下面。下面是目录结构的框架：

    # 工程顶层目录
    xwalk-echo-project/

      # 插件目录
      xwalk-echo-extension-src/
        build/
          ...temporary build artefacts...
        java/
          ...Java source files for the extension...
        js/
          ...JavaScript for the extension...
        lib/
          ...third party jar files (installed via Ivy)...
        tools/
          ...jar files to assist with the build...
        xwalk-echo-extension/
          ...temporary output directory for the extension...
        build.xml                     # Ant build file
        ivy.xml                       # Ivy configuration for Ant
        xwalk-echo-extension.json     # extension configuration

      # web应用目录
      xwalk-echo-app/
        assets/
          ...images, stylesheets etc...
        js/
          ...JavaScript files...
        icon.png                      # application icon
        index.html                    # main HTML file
        manifest.json                 # Crosswalk manifest

虽然您是在web应用的同级目录下开发插件，但是记住Crosswalk插件是可以在多个工程之间复用的。

### 创建工程目录

简单起见，假设您已经有有一个工程文件夹`~/<my projects directory>`。如下所示，在您的工程文件夹下创建教程中的目录：

    cd ~/<my projects directory>

    # set up top-level directory for the xwalk-player project
    mkdir xwalk-echo-project
