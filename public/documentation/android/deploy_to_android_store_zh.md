# 部署到安卓商店

完成以上章节之后，Crosswalk app就可以正式发布了。发布的方式取决于使用的打包方式(嵌入模式 或者 共享模式)。

本教程中，使用的是嵌入模式进行打包的。然而，用户也可能[使用共享模式打包](/documentation/android/run_on_android.html#shared-vs-embedded-mode)。

下面将会简要介绍下如何发布俩种打包方式下的apk。

## 发布嵌入模式Crosswalk app

为了保证Crosswalk app能运行在各种安卓机器上，建议开发者上传所有使用嵌入模式打包的app。实际上，这需要开发者为一个嵌入模式下的web app做下面的事情。

*   上传x86版本的Crosswalk app到安卓商店。
*   上传ARM版本的Crosswalk app到安卓商店。

谷歌应用商店支持同一个app有多个apk可以上传，详情参考[文章](http://developer.android.com/google/play/publishing/multiple-apks.html)。

## 发布共享模式Crosswalk app

对于使用共享模式打包的web app，建议开发者：

*   上传x86版本的Crosswalk runtime apk到安卓商店。对于所有使用共享模式的app，只需要上传一次Crosswalk runtime apk。
*   上传基于ARM的Crosswalk runtime apk到安卓商店。对于所有使用共享模式的app，只需要上传一次Crosswalk runtime apk。
*   上传所有基于Crosswalk的app到安卓商店。在不同平台下，这些app会共享相应版本的runtime apk。

谷歌应用商店已经支持一个app可以存在多个apk包（这里的Crosswalk runtime就是这样的一种"app"，它被用来运行您的*真正的*web应用）：详情参见[文章](http://developer.android.com/google/play/publishing/multiple-apks.html)。
