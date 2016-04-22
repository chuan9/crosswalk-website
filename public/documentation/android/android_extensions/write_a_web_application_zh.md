# 编写web应用

本教程的web应用是一个简单的HTML5/CSS/Javascript应用，它包含一些其他文件以便可以用在Crosswalk包中。关于Crosswalk 应用的详细细节，请参考[*开始页面*](/documentation/android/build_an_application.html)。

1.  为web应用创建一个目录：

        cd ~/<my projects directory>/xwalk-echo-project

        # web应用的顶层目录
        mkdir xwalk-echo-app
        cd xwalk-echo-app

        # 创建应用组件的目录
        mkdir -p assets js

    在添加文件到`xwalk-echo-app`目录后，进行下面的步骤。

2.  添加`manifest.json`文件

        {
          "name": "xwalk_echo_app",
          "xwalk_version": "0.0.0.1",
          "start_url": "index.html",
          "icons": [
            {
              "src": "icon.png",
              "sizes": "96x96",
              "type": "image/png",
              "density": "4.0"
            }
          ]
        }

3.  添加图标文件`icon.png`。

    上面的manifest文件指定了一个大小为96x96px，名为`icon.png`的图片作为应用的图标。Crosswalk使用它作为应用的图标，将会出现在安卓应用列表/主屏幕上等。

    您可以使用自己的图片(正确尺寸)，或者参考[页面](/documentation/android/build_an_application.html#A-simple-application)拷贝Crosswalk默认的图标。

4.  添加HTML文件`index.html`：

        <!DOCTYPE html>
        <html>
        <head>
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta charset="utf-8">
        <title>xwalk-echo-app</title>
        <link rel="stylesheet" href="assets/base.css">
        </head>
        <body>

        <header>
          <h1>xwalk-echo-app</h1>
        </header>

        <script src="js/main.js"></script>
        </body>
        </html>

5.  在`assets/base.css`下添加简单的CSS文件：

        body {
          font-size: 1.5em;
          margin: 0;
          font-family: sans-serif;
        }

        h1 {
          width: 100%;
          text-align: center;
          margin: 0.5em 0;
        }

        header {
          border-bottom: 0.25em solid black;
        }

    这里加大了字体并添加了一些线条和空格，作用是使布局稍显优美。

6.  添加JavaScript文件`js/main.js`。这个文件中的代码使用了Crosswalk插件反射信息，并在新的`<p>`元素中显示结果：

        document.addEventListener('DOMContentLoaded', function () {
          echo.echoAsync('Hello world').then(
            function (result) {
              var p = document.createElement('p');
              p.innerHTML = result.content;
              document.body.appendChild(p);
            }
          );
        });

    添加到`DOMContentLoaded`事件中的listener是这段代码的核心。这里负责调用插件API，调用是通过异步方法`echoAsync()`，并且添加了处理返回的Promise的handlers。插件返回对象的`content`属性被用来作为一个新的段落，这个段落添被添加到文档中。

现在web应用和插件都已经准备好，您可以创建apk文件并在安卓上运行了。
