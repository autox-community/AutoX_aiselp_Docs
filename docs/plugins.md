# 插件

auto.js 提供了加载插件的机制，允许用户编写带有 Activity, Service, C/C++库等的 apk，安装到 Android 设备上，并用 Auto.js 加载和调用。

一个插件是一个可独立安装的 apk 文件，用户安装后，再通过 `plugins` 模块加载插件和调用其中的 API。

安装后可以使用以下代码来加载插件

```js
// packageName 为插件的包名，如果没有安装该插件则会抛出错误
var plugin = plugins.load(packageName);
```

:::warning
由于插件系统的设计缺陷，插件 APP 的目标 SDK 与 Autojs 不一致则可能出现兼容性异常
:::

## 开发一个插件

:::info
为了获得最佳兼容性，可将插件 APP 的目标 SDK 与最小 SDK 设置与 Autojs 一致
:::

### 插件 SDK 集成

1. 新建一个 Android 项目，在项目的 build.gradle 文件中添加：

```groovy
allprojects {
    repositories {
        // ...
        maven { url 'https://jitpack.io' }
    }
}
```

2. 在具体模块(比如 app)的 build.gradle 文件中添加：

```groovy
dependencies {
    // ...
    implementation 'com.github.hyb1996:Auto.js-Plugin-SDK:0.2'
}
```

### 插件配置

1. 新建 PluginHelloWorld 文件，继承于 Plugin.

```java
public class PluginHelloWorld extends Plugin {

    public PluginHelloWorld(Context context, Context selfContext, Object runtime, Object topLevelScope) {
        super(context, selfContext, runtime, topLevelScope);
    }

    // 返回插件的JavaScript胶水层代码的assets目录路径
    @Override
    public String getAssetsScriptDir() {
        return "plugin-helloworld";
    }

    // 插件public API，可被JavaScript代码调用
    public String getStringFromJava() {
        return "Hello, Auto.js!";
    }

    // 插件public API，可被JavaScript代码调用
    public void say(String message) {
        getSelfContext().startActivity(new Intent(getSelfContext(),  HelloWorldActivity.class)
                .putExtra("message", message)
                .addFlags(Intent.FLAG_ACTIVITY_NEW_TASK));
    }
}
```

2. 新增 MyPluginRegistry 文件，继承于 PluginRegistry:

```java
public class MyPluginRegistry extends PluginRegistry {

    static {
        // 注册默认插件
        registerDefaultPlugin(new PluginLoader() {
            @Override
            public Plugin load(Context context, Context selfContext, Object runtime, Object topLevelScope) {
                return new PluginHelloWorld(context, selfContext, runtime, topLevelScope);
            }
        });
    }
}
```

在 AndroidManifest.xml 中配置以下 meta-data, name 为"org.autojs.plugin.sdk.registry"，value 为 MyPluginRegistry 的包名。

```xml {4-6}
  <application
        ...>

        <meta-data
            android:name="org.autojs.plugin.sdk.registry"
            android:value="org.autojs.plugin.sdk.demo.MyPluginRegistry" />

        <activity
            android:name=".HelloWorldActivity"
            android:exported="true" />

        <service
            android:name=".HelloworldPluginService"
            android:exported="true" />
    </application>
```

3. 编写 JavaScript 胶水层

在 assets 的相应目录(由 Plugin.getAssetsScriptDir 返回)中添加 index.js 文件，用于作为胶水层导出插件 API。

```js
module.exports = function (plugin) {
  let runtime = plugin.runtime;
  let scope = plugin.topLevelScope;

  function helloworld() {}

  helloworld.stringFromJava = plugin.getStringFromJava();

  helloworld.say = function (message) {
    plugin.say(message);
  };

  return helloworld;
};
```

4. 在 Auto.js 中调用
   编译插件为 apk(assembleDebug/assembleRelease)，安装到设备上。在 Auto.js 中使用以下代码调用：

```js
let helloworld = $plugins.load("org.autojs.plugin.sdk.demo");
console.log(helloworld.stringFromJava);
helloworld.say("Hello, Auto.js Pro Plugin");
```
