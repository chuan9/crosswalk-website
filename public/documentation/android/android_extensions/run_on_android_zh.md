# 在安卓上运行

Building a Crosswalk Android application with extensions is almost identical to building any other Crosswalk Android application, and uses the same `make_apk.py` script (see the [*Run on Android* documentation](/documentation/android/run_on_android.html)). The main difference is that an additional `--extensions` option is required, to instruct the script where to find extension directories.

## 生成安卓应用程序包

To make Android packages for your project, do the following:

1.  Build the extension, as described [previously](/documentation/android/android_extensions/write_an_extension.html#Build-the-extension). This will create the `xwalk-echo-extension/` directory with the extension files inside.

2.  After the build, the extension project directory (`xwalk-echo-extension-src/`) will contain a `lib/crosswalk-${XWALK-STABLE-ANDROID-X86}` directory, which is an unpacked Crosswalk Android distribution. You can use the scripts in here to build the Android packages.

    Follow the [system setup page](/documentation/android/system_setup.html) to installed the Android SDK. Then use these commands to create the packages:

    * Set an environment variable to the location of the project

            > PROJECT_DIR=~/<my projects directory>/xwalk-echo-project

    * Enter the unpacked Crosswalk Android directory

            > cd $PROJECT_DIR/xwalk-echo-extension-src/lib/crosswalk-${XWALK-STABLE-ANDROID-X86}

    * Invoke the package builder
    
            > python make_apk.py --package=org.crosswalkproject.example \
              --enable-remote-debugging --fullscreen \
              --manifest=$PROJECT_DIR/xwalk-echo-app/manifest.json \
              --extensions=$PROJECT_DIR/xwalk-echo-extension-src/xwalk-echo-extension/

    The generated packages are in the `~/<my projects directory>/xwalk-echo-project/xwalk-echo-extension-src/lib/crosswalk-${XWALK-STABLE-ANDROID-X86}` directory, and called `xwalk_echo_app_x86.apk` and `xwalk_echo_app_arm.apk`.

    Note that if you are using multiple extension directories, you still have a single `--extensions` option but supply the paths as a comma-delimited string, e.g. `--extensions=myextension1,myextension2`

## 安装程序

选择适合您机器架构的apk并使用如下命令安装：

    $ adb install -r ~/<my projects directory>/xwalk-echo-project/xwalk-echo-extension-src/lib/crosswalk-${XWALK-STABLE-ANDROID-X86}/xwalk_echo_app_x86.apk

命令将会安装x86版本的apk。更多关于安装apk的细节，参考[页面](/documentation/android/run_on_android.html)。

应用的图标将会出现在安卓屏幕的应用列表中。点击图标您将看到屏幕上出现类似下面的内容：

![Crosswalk Android application with extensions on x86 ZTE Geek](/assets/android-extensions-x86.png)

(上面的截图来自ZTE x86 Geek上的程序)

应用会展示内容"You said: Hello world"，证明了Crosswalk插件中的Java模块和JavaScript模块通信正常。
