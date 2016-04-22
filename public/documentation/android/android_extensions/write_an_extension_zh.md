# 编写插件

Crosswalk插件是用Java编写的，因此插件可以使用标准的[Android APIs](http://developer.android.com/reference/packages.html)。

此外，因为安卓上的Crosswalk是一个[Activity](http://developer.android.com/reference/android/app/Activity.html)并且可以获取[Context](http://developer.android.com/reference/android/content/Context.html)，因此插件也可以使用这些对象。然而，本教程中插件没有使用任何Android APIs，因为它仅仅做了一个字符串的操作。

虽然有大量的Android APIs可以使用，但是在某些情况下可能依然需要在插件中使用第三方库。本教程用[Gson](https://code.google.com/p/google-gson/)读写JSON为例，阐述如何使用第三方插件。 (虽然Android API有读写JSON的部分能力，但是使用Gson更加简洁)

## 插件是怎么做的？

下面是插件工作的流程图：

<ol>
  <li>
  当附有插件的Crosswalk应用在安卓设备上启动之后，<code>Echo</code>插件的一个实例将被初始化。
  </li>

  <li>
    <p>应用的web部分将会调用一个方法，这个方法是由插件中Javascript模块暴漏出来的。<code>echo.echoAsync()</code> (异步的)或者 <code>echo.echo()</code> (同步的)。两个方法都需要一个参数，这个参数代表将要显示的信息。</p>

     <p>在上面的两个方法中，插件的JavaScript模块将会创建如下结构的message对象：</p>

<pre><code>{
  "id": "1",
  "content": "Hello world"
}</code></pre>

    <p>每条消息包含一个唯一的请求ID。当插件返回结果时，这个ID是保证返回的结果对应相应的回调函数。</p>

    <p>message对象被序列化成JSON格式并且被发送到插件的Java模块。</p>
  </li>

  <li>
    在<code>Echo</code>实例里(Java)，从插件的JavaScript模块接收的JSON字符串被反序列化成了<code>message</code>对象。
  </li>

  <li>
    接下来，<Echo>实例中的私有方法<code>echo()</code>被调用。注意这个方法接收了请求ID(从JavaScript那边)。<code>echo()</code>简单地给字符串加上前缀并返回。
  </li>

  <li>
    <p><code>Echo</code>实例创建一个<code>message</code>对象。在返回最初始的调用者之前，它被序列化成JSON格式的字符串(通过Gson)。</p>

    <p>JSON格式的字符串被立即返回(同步情况下)或者作为一条消息被发送到插件的JavaScript模块(异步情况下)。</p>
  </li>

  <li>
    <p><a name="message-data-structure"></a>在JavaScript方面，JSON字符串被反序列化成如下格式的对象：</p>

    <pre><code>{
  "id": "1",
  "content": "You said: Hello world"
}</code></pre>

    <p>Web应用可以利用上面的对象做很多事情；在本教程中，它创建了一个DOM元素并把带有前缀的内容显示出来。</p>
  </li>
</ol>

在下面的内容中，您将创建插件的Java和JavaScript模块并用配置文件把他们写在一起。

<h2 id="Set-up-the-directory-structure">创建目录</h2>

    cd ~/<my projects directory>/xwalk-echo-project

    # 插件的顶层目录
    mkdir xwalk-echo-extension-src
    cd xwalk-echo-extension-src

    # 这是插件的Java代码；注意Java的类
    # 将会在包org.crosswalkproject.sample里面。
    # - 可以在您的工程中改变参数名
    mkdir -p java/org/crosswalkproject/sample

    # 这是插件的JavaScript代码
    mkdir js

    # 工程中第三方插件
    mkdir tools

`build/`, `lib/` and `xwalk-echo-extension/`目录([工程概览](/documentation/android_extensions/host_and_target_setup_zh.html#project-outline)中展示的)将会在编译的时候自动创建。

注意：请在`xwalk-echo-extension-src/`(插件的顶层目录)目录下进行下面的操作。

## 为插件添加Java代码

首先，创建一个继承[`XWalkExtensionClient`](https://github.com/crosswalk-project/crosswalk/blob/master/app/android/runtime_client/src/org/xwalk/app/runtime/extension/XWalkExtensionClient.java)的Java类。

`java/org/crosswalkproject/sample/Echo.java`:

    package org.crosswalkproject.sample;

    import org.xwalk.app.runtime.extension.XWalkExtensionClient;
    import org.xwalk.app.runtime.extension.XWalkExtensionContextClient;
    import com.google.gson.Gson;

    public class Echo extends XWalkExtensionClient {
      private Gson gson = new Gson();

      public Echo(String name, String jsApiContent, XWalkExtensionContextClient xwalkContext) {
        super(name, jsApiContent, xwalkContext);
      }

      private String echo(String requestJson) {
        Message request = gson.fromJson(requestJson, Message.class);
        String reply = "You said: " + request.content;
        Message response = new Message(request.id, reply);
        return gson.toJson(response);
      }

      @Override
      public void onMessage(int instanceId, String requestJson) {
        postMessage(instanceId, echo(requestJson));
      }

      @Override
      public String onSyncMessage(int instanceId, String requestJson) {
        return echo(requestJson);
      }
    }

类中有两个关键的方法，这两个方法重写了`XWalkExtensionClient`类中的方法。它们提供了插件中JavaScript模块和Java模块通信的渠道。

*   `onMessage()`: 适用于异步的消息。
*   `onSyncMessage()`: 适用于同步的消息。

两个方法的内容都调用了`echo()`方法，这个方法：

1.  将原始的请求参数反序列化成`Message`对象(参见下面)。
2.  在字符串前面加上"You said: "的前缀。
3.  创建一个`Message`对象,这个对象包含了带有前缀的字符串和原始的请求ID。
4.  序列化成JSON格式。

然而，他们返回结果的方式不同：

*   `onSyncMessage()`直接返回JSON格式的字符串到调用它的JavaScript代码。
*   `onMessage()`通过调用`postMessage()`(`XWalkExtensionClient`中的方法)间接返回结果。它异步地返回JSON格式的字符串到JavaScript模块中的API中，JavaScript API可以监听并处理JSON格式的字符串。

建议使用下面的类，可以更加容易地对JSON格式进行操作：

`java/org/crosswalkproject/sample/Message.java`:

    package org.crosswalkproject.sample;

    public class Message {
      public String id;
      public String content;

      public Message(String id, String content) {
        this.id = id;
        this.content = content;
      }
    }

(像上面的`Message`类中一样，对于Java对象中的公共属性变量，Gson可以对它们进行序列化和反序列化操作。)

在编译插件之前，您需要像像下面一样，添加其他必要的文件。

<h2 id="Add-the-extension-configuration-file">添加插件配置文件</h2>

配置文件告诉Crosswalk打包工具插件的Java和JavaScript模块是如何一起工作的。

创建一个含有下面内容的JSON文件`xwalk-echo-extension.json`：

    {
      "name":  "echo",
      "class": "org.crosswalkproject.sample.Echo",
      "jsapi": "xwalk-echo-extension.js",
      "permissions": []
    }

文件中对象的各个部分解释如下：

*   `name`: 插件的命名空间，这个名字的作用域是web应用的全局空间。例如，插件的命名空间是`echo`,那么在web应用中就可以像下面那样使用：

        // 异步的
        echo.echoAsync("Hello world").then(
          function (result) {
            // ...process result...
          }
        );

        // 同步的
        var message = echo.echo("Hello world");

    注意在web应用中不需要导入JavaScript文件：当插件被实例化之后，插件的API会自动在Javascript全局作用域中变得可用。可用的API是在下面即将创建的`xwalk-echo-extension.js`文件中定义的。

*   `class`: 实现插件的Java类：在这个例子中，是类 `org.crosswalkproject.sample.Echo`。注意名字应该包括包名和类名。

*   `jsapi`: 定义JavaScript API的文件。您将在下面的章节中创建它。

*   `permissions`: 插件需要的其它权限。由于教程中的插件仅仅需要Crosswalk默认权限，因此这里是空的。如果您在开发自己的应用，可能需要添加其它权限。

    `permissions`中的字符串应该匹配相应的安卓权限；详情参见[安卓权限列表](http://developer.android.com/reference/android/Manifest.permission.html)。例如，如果需要获取`FLASHLIGHT`和`GET_ACCOUNTS`权限，插件配置文件中的`permissions`属性应具有如下字样：

        "permissions": ["android.permission.FLASHLIGHT", "android.permission.GET_ACCOUNTS"]

在编译的时候，`make_apk.py`将所有插件中的配置文件整合成一个`extensions-config.json`文件。这是Crosswalk真正用来加载插件的类以及类对应的JavaScript API。

<h2 id="Add-the-JavaScript-API-file">添加JavaScript API文件</h2>

创建具有如下内容的`js/xwalk-echo-extension.js`：

    /*
    echoAsync()和echo()解析/返回具有如下格式的对象(echoAsync对应解析操作，echo对应返回操作)

    {
      id: '<请求ID>',
      content: '<从插件的Java模块返回的内容>'
    }
    */

    // 为插件的每个调用提供一个唯一的ID
    var counter = 0;

    // 匹配从请求ID到相应回调的数组
    var successCbs = {};

    // 私有方法。用来创建message对象并把它转换成可以传递到插件Java模块的JSON字符串。
    var messageToJson = function (counter, message) {
      var obj = {
        id: '' + counter,
        content: message
      };

      return JSON.stringify(obj);
    };

    // 所有信息的message监听器；根据消息中的ID，调用正确的回调函数
    extension.setMessageListener(function (message) {
      var data = JSON.parse(message);
      var cb = successCbs[data.id];

      if (cb) {
        cb(data);
        delete successCbs[data.id];
      }
    });

    // 返回一个promise：解析file对象的一个数组，或者当调用插件失败的时候，
    // 返回一个错误
    exports.echoAsync = function (message) {
      counter += 1;
      var messageJson = messageToJson(counter, message);

      return new Promise(function (resolve, reject) {
        successCbs[counter] = resolve;

        // 注意必须向postMessage()中传递一个字符串
        try {
          extension.postMessage(messageJson);
        }
        catch (e) {
          reject(e);
        }
      });
    };

    // 返回经过插件处理过的对象
    exports.echo = function (message) {
      counter += 1;
      var messageJson = messageToJson(counter, message);

      // 注意必须向sendSyncMessage()中传递一个字符串
      var result = extension.internal.sendSyncMessage(messageJson);

      return JSON.parse(result);
    };

文件中需要注意的几点：

*   每次调用，`counter`变量都会加1，并且在插件Java模块中，作为请求ID出现。这样就保证从插件返回的消息可以被相应的回调函数处理。为了一致性（无论同步还是异步，插件的java模块可以总是接收含有`id`和`content`的JSON字符串），counter变量也被传递到方法`sendSyncMessage()`中，虽然没有必要使同步的方法与回调函数保持一致（因为同步中的结果是立即返回的）。

*   当调用`echoAsync()`时，会创建一个新的Promise（可以参照下一小节）。Promise中成功的回调函数(`resolve`)与唯一的ID绑定在一起，这是因为这个请求被添加到了`successCbs`中。

*   `extension.setMessageListener()`设置了一个回调函数，当有消息从插件的Java模块中返回时，就会触发这个回调函数。*每个*消息返回时，都会触发这个回调函数；但是实际上只会处理那些已经存储在`successCbs`对象中的数据，`successCbs`是一个从请求ID到handlers的映射。当接收到消息时，就会从`successCbs`中查找正确的handler并且作为一个参数被调用。在handler被调用之后，就会从`successCbs`中删除。

*   插件中任何想要作为JavaScript API被暴漏的属性(方法/对象/常量等等)应该添加到JavaScript API文件中的`exports`对象中。这种做法类似nodejs模块中的`exports`对象，作用是定义公共接口。其他*没有*添加到`exports`对象中的变量，只对本文件可见，所以不会污染整个web应用的作用域。

    JavaScript暴漏的命名空间(`echo`)是在插件配置文件中设置的，也就是上文中创建的`xwalk-echo-extension.json`。

*   插件Java模块是可以通过`extension`对象获取的。注意上面的代码调用了函数`extension.internal.sendSyncMessage()`和`extension.postMessage()`，这两个函数可以和上面的Java代码通信。

### Promises, promises

在插件的JavaScript API文件中定义的同步方法(`echo()`)是直接的，并有下面的签名：

    echo.echo() : Message

Message定义在[这里](#message-data-structure).

相比之下，异步方法`echoAsync()`有如下的签名：

    echo.echoAsync() : Promise

如果您不熟悉[Promises](http://promises-aplus.github.io/promises-spec/) (一个相对较新的web应用开发工具)，这看起来会有些怪。为什么不直接用回调函数？例如用具有如下签名的异步方法替代返回的Promise：

    echo.echoAsync(callback) : undefined

当Java模块的API返回`Message`对象(已经序列化成JSON)时调用`callback(message)`。

回调函数的问题在于管理：如果回调函数发生多层嵌套，就会导致所谓的[厄运金字塔](http://survivejs.com/common_problems/pyramid.html)。例如，假设您想要为返回结果添加一些额外的属性，您需要操作多个对象，并且当最初是的结果返回给回调函数时，调用这些对象的方法：

    echo.echoAsync(function (result) {

      // 为result添加更多属性
      decorator1.decorate(result, function (decoratedResult) {

        // 为decoratedResult添加更多的属性
        decorator2.decorate(decoratedResult, function (evenMoreDecoratedResult) {
          displayData(evenMoreDecoratedResult);
        });

      });

    });

在上面的例子中，有两个异步函数`decorator1.decorate()`和`decorator2.decorate()`，它们为返回结果添加额外的属性。注意金字塔已经开始出现。添加更多的回调会使情况更加糟糕。

还有其它避免出现金字塔的方法；但是使用Promises好处在于在顶部添加这些方法，简化了异步的代码和回调嵌套。例如，如果`echoAsync()`返回一个Promise，并且`*.decorate()`也返回一个Promises，我们可以使用下面的代码：

    echo.echoAsync()
    .then(
      function (result) {
        return decorator1.decorate(result);
      }
    )
    .then(
      function (decoratedResult) {
        return decorator2.decorate(decoratedResult);
      }
    )
    .then(displayData);

当`echoAsync()`返回的Promise的值是*resolved*（就是它完成了异步的操作并且值不是错误的时候），方法`then()`使用一个函数来处理`*.decorate()`的Promise返回的值。当Promise的值为*rejected*时（也就是Promise失败后值变为错误），也可以使用一个函数来处理Promise返回的所有的错误。
您可以看出来上面的代码避免了金字塔的出现，并且处理数据的步骤也变得很清晰。

注意最后一个`then()`调用，因为我们不返回任何值，只是传入了函数`displayData`，并把`decorator2.decorate()`的结果隐式地传递给`displayData`。

在许多环境里，Promises是不被支持的；典型的方案就是使用其他库像[Q](http://documentup.com/kriskowal/q/)去填补缺口。相比之下，在Crosswalk里面，您已经有支持Promises的环境，就是您正在使用的插件中JavaScript模块；因此不需要额外的库。

## 添加编译基础框架

在Crosswalk应用中使用插件，您必须在安卓包中包括它。Crosswalk打包工具对于包中插件的组织架构有严格的要求。插件的组织架构*必须*像下面这个样子：

    myextension/
      myextension.jar
      myextension.js
      myextension.json

所有的名字*必须*匹配：目录名字必须匹配`.jar`， `.js`和`.json`这些文件的前缀，否则插件将不会被package包含。

您需要用自己的插件名字替换"myextension"。对于教程中的插件，目录如下：

    xwalk-echo-extension/
      xwalk-echo-extension.jar
      xwalk-echo-extension.js
      xwalk-echo-extension.json

然而，目前为止，在实际情况中您可能注意到文件并没有和上面的布局文件一样(也就是说`xwalk-echo-extension.js`在`js`目录，没有`.jar`文件)。这就是为什么需要编译基础框架。您将设置一个自动化的编译，它创建一个临时目录 `xwalk-echo-extension`并复制编译三个必须文件到目录中，而不是手动的进行上面的步骤。

Ivy和Ant是Java工程中常用的工具，因此您将使用它们来编译插件：在编译时使用Ivy下载Gson jar文件(由于`Echo`依赖于它)；使用Ant编译插件的Java代码并拷贝文件到指定路径。(如果您对Android Studio熟悉，也可以用Android Studio，而不必用command-line工具。)

接下来两小节将会叙述如何使用Ivy和Ant。

### 配置Ivy

遵循下面的步骤来安装和配置Ivy：

1.  下载Apache Ivy发行版。可以从[Apache Ivy下载](https://ant.apache.org/ivy/download.cgi)。例如，下载Ivy 2.4.0-rc1：

        $ wget http://www.mirrorservice.org/sites/ftp.apache.org/ant/ivy/2.4.0-rc1/apache-ivy-2.4.0-rc1-bin.zip

2.  解压并拷贝Ivy jar文件到`tools/`目录下：

        $ unzip apache-ivy-2.4.0-rc1-bin.zip
        $ cp apache-ivy-2.4.0-rc1/ivy-2.4.0-rc1.jar tools/

    解压完成之后，可以删除zip包。

3.  添加Ivy配置文件，`ivy.xml`到顶层目录：

        <?xml version="1.0" encoding="UTF-8"?>
        <ivy-module version="2.3">
          <info organisation="org.crosswalkproject.sample" module="xwalk-echo-extension" />
          <dependencies>
            <dependency org="com.google.code.gson"
                        name="gson"
                        rev="2.2.4"
                        conf="default->master" />
          </dependencies>
        </ivy-module>

    这里您只需一个依赖包(`gson.jar`)，但是您可以添加其他第三方库到这个文件中。

    注意Crosswalk不在Ivy仓库中，它将在主要编译文件中用Ant下载。

<h3 id="Add-an-Ant-buildfile">添加Ant编译文件</h3>

您应该已经按照[系统配置页面](/documentation/android/system_setup_zh.html)的步骤安装了Ant。

Ant安装之后，添加一个具有以下内容的编译文件：`build.xml`到工程的顶层目录：

    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns:ivy="antlib:org.apache.ivy.ant" name="xwalk-echo-extension" default="dist">
      <!-- Java源码 -->
      <property name="src" value="java" />

      <!-- 下载第三方库 -->
      <property name="lib" value="lib" />

      <!-- Crosswalk安卓版本 -->
      <property name="crosswalk-version" value="${XWALK-STABLE-ANDROID-X86}" />

      <!-- Crosswalk安卓的URL -->
      <property name="crosswalk-download-url"
                value="https://download.01.org/crosswalk/releases/crosswalk/android/stable/${crosswalk-version}/crosswalk-${crosswalk-version}.zip" />

      <!-- Crosswalk安卓文件的下载路径 -->
      <property name="crosswalk-zip" value="${lib}/crosswalk.zip" />

      <!-- 临时编译目录 -->
      <property name="build" value="build" />

      <!-- 编译插件的最终位置 -->
      <property name="dist" value="xwalk-echo-extension" />

      <!--  包含Ivy Ant任务jar文件的classpath -->
      <path id="ivy.lib.path">
        <fileset dir="tools" includes="*.jar"/>
      </path>

      <!-- 删除+创建临时编译目录 -->
      <target name="prepare">
        <delete dir="${build}" quiet="true" />
        <delete dir="${dist}" quiet="true" />

        <mkdir dir="${build}" />
        <mkdir dir="${lib}" />
        <mkdir dir="${dist}" />
      </target>

      <!-- 使用Ivy下载依赖库 -->
      <target name="download-deps" depends="prepare">
        <taskdef resource="org/apache/ivy/ant/antlib.xml"
                 uri="antlib:org.apache.ivy.ant"
                 classpathref="ivy.lib.path" />
        <ivy:retrieve pattern="${lib}/[artifact]-[revision].[ext]" />
      </target>

      <!-- 检查Crosswalk zip包文件是否存在 -->
      <target name="check-crosswalk-present" depends="prepare">
        <available file="${crosswalk-zip}" property="crosswalk-zip.present"/>
      </target>

      <!-- 如果不在重新获取crosswalk.zip -->
      <target name="download-crosswalk" depends="prepare, check-crosswalk-present"
              unless="crosswalk-zip.present">
        <!-- fetch from the download site -->
        <get src="${crosswalk-download-url}" dest="${crosswalk-zip}" />

        <!-- 解压到lib/crosswalk-*/ -->
        <unzip src="${crosswalk-zip}" dest="${lib}" />
      </target>

      <!-- 编译插件的Java代码 -->
      <target name="compile" depends="download-deps, download-crosswalk">
        <first id="app_runtime_java">
          <fileset dir="${lib}/crosswalk-${crosswalk-version}" includes="**/xwalk_app_runtime_java.jar" />
        </first>
        <javac srcdir="${src}" destdir="${build}"
               encoding="utf-8" debug="true" verbose="true">
          <classpath>
            <fileset dir="${lib}" includes="*.jar" />
            <file file="${toString:app_runtime_java}" />
          </classpath>
        </javac>
      </target>

      <!--
      打包第三方库的代码和插件代码到单独的
      jar中，拷贝支持文件到xwalk-echo-extension/
      目录下，注意我们不需要打包任何Crosswalk jar包，因为它们
      将会通过打包工具添加进去；同时我们不需要android.jar，
      因为在安卓目标机中已经存在
      -->
      <target name="dist" depends="compile">
        <unjar dest="${build}">
          <fileset dir="${lib}">
            <include name="*.jar" />
          </fileset>
        </unjar>

        <jar destfile="${dist}/xwalk-echo-extension.jar">
          <fileset dir="${build}" excludes="META-INF/**" />
        </jar>

        <copy file="xwalk-echo-extension.json" todir="${dist}" />
        <copy file="js/xwalk-echo-extension.js" todir="${dist}" />
      </target>
    </project>

对于小型工程，上面是一个相当标准的Ant编译文件。默认任务是`发行版`，它将会做下面的事情：

<ol>
  <li>删除并重新创建<code>build/</code>和<code>xwalk-echo-extension/</code>目录。</li>

  <li>下载依赖包Gson并把它放入目录<code>lib/</code>中(通过Ivy).</li>

  <li>下载Crosswalk安卓(通过HTTP)并解压到<code>lib/</code>目录下。注意如果您想使用<a href="/documentation/downloads">beta或者canary版本的Crosswalk</a>，您将需要修改<code>crosswalk-version</code>和<code>crosswalk-download-url</code> <code>&lt;property&gt;</code>元素如下：

    <ul>
      <li>
        <p><strong>beta:</strong></p>

        <pre><code>&lt;property name="crosswalk-version" value="${XWALK-BETA-ANDROID-X86}" /&gt;
&lt;property name="crosswalk-download-url"
          value="https://download.01.org/crosswalk/releases/crosswalk/android/beta/${crosswalk-version}/crosswalk-${crosswalk-version}.zip" /&gt;</code></pre>
      </li>

      <li>
        <p><strong>canary:</strong></p>

        <pre><code>&lt;property name="crosswalk-version" value="...Crosswalk canary version..." /&gt;
&lt;property name="crosswalk-download-url"
          value="https://download.01.org/crosswalk/releases/crosswalk/android/canary/${crosswalk-version}/crosswalk-${crosswalk-version}.zip" /&gt;</code></pre>

        <p>您可以通过<a href="/documentation/downloads">下载页面</a>找到当前的canary版本</p>
      </li>
    </ul>
  </li>

  <li>在<code>src/</code>目录下编译插件的Java源代码, 把编译出来的<code>.class</code>文件放到<code>build/</code>目录下</li>

  <li>解包Gson夹包到<code>build/</code>目录。这样它就可以被包含在插件的夹包文件中。</li>

  <li>在<code>xwalk-echo-extension/</code>下面创建一个夹包，这个夹包包含插件的<code>.class</code>文件和从Gson夹包中解包出来的文件。</li>

  <li>拷贝插件JSON配置文件<code>xwalk-echo-extension.json</code>和JavaScript API定义文件<code>js/xwalk-echo-extension.js</code>到<code>xwalk-echo-extension/</code>目录下。</li>

</ol>

最终的输出是，在`xwalk-echo-extension/`目录下，包含了插件的所有文件，这些文件即将被包含在Crosswalk `.apk`文件，也就是：

    xwalk-echo-extension/
      xwalk-echo-extension.jar
      xwalk-echo-extension.js
      xwalk-echo-extension.json

<h2 id="Build-the-extension">编译插件</h2>

在添加标准的Ant编译文件之后，编译插件变得非常简单，在`xwalk-echo-extension-src/`目录下执行：

    $ ant

这将执行`发行版`任务(参考上面)。(由于第一次编译需要下载第三方的依赖库，可能会需要一点时间)

插件编译完成之后，下一步就是创建使用插件的web应用。
