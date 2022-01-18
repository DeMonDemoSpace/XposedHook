### Xposed-Hook

Android获取敏感信息整改---基于Xposed的方法检测。

#### 1.安装Xposed虚拟机

下载页面：[VirtualXposed](https://github.com/android-hacker/VirtualXposed/releases)

需要注意的是：VirtualXposed再0.18.2版本之后只支持64位的App。
如果你的App目前还是32位的，只能使用[32位 VirtualXposed_0.18.2.apk](https://github.com/android-hacker/VirtualXposed/releases/download/0.18.2/VirtualXposed_0.18.2.apk)

下载后安装到手机待用。

#### 2.基于Xposed库开发自定义模块

可以直接参考本库代码[XposedHook.java](https://github.com/DeMonDemoSpace/XposedHook/blob/master/app/src/main/java/com/demon/xposed_hook/XposedHook.java)，修改后编译运行即可。

也可以参照下面的方法自己新建项目后，自定义。


##### 1. 引入Xposed库

```
   compileOnly 'de.robv.android.xposed:api:82'
   compileOnly 'de.robv.android.xposed:api:82:sources'
```


##### 2.自定义Xposed模块代码

如下我们使用```XposedHelpers.findAndHookMethod```检测获取MAC的方法：```getMacAddress```&```getHardwareAddress```，并输出日志。

```java

public class XposedHook implements IXposedHookLoadPackage {
    private static final String TAG = "HookLogin";

    public void handleLoadPackage(XC_LoadPackage.LoadPackageParam lpparam) {
        if (lpparam == null) {
            return;
        }

        Log.e(TAG, "Load app packageName:" + lpparam.packageName);
        XposedHelpers.findAndHookMethod(
                android.net.wifi.WifiInfo.class.getName(),
                lpparam.classLoader,
                "getMacAddress",
                new XC_MethodHook() {
                    @Override
                    protected void beforeHookedMethod(MethodHookParam param) {
                        XposedBridge.log("调用getMacAddress()获取了mac地址");
                    }

                    @Override
                    protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                        XposedBridge.log(getMethodStack());
                        super.afterHookedMethod(param);
                    }
                }
        );

        XposedHelpers.findAndHookMethod(
                java.net.NetworkInterface.class.getName(),
                lpparam.classLoader,
                "getHardwareAddress",
                new XC_MethodHook() {
                    @Override
                    protected void beforeHookedMethod(MethodHookParam param) {
                        XposedBridge.log("调用getHardwareAddress()获取了mac地址");
                    }

                    @Override
                    protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                        XposedBridge.log(getMethodStack());
                        super.afterHookedMethod(param);
                    }
                }
        );

    }

    private String getMethodStack() {
        StackTraceElement[] stackTraceElements = Thread.currentThread().getStackTrace();

        StringBuilder stringBuilder = new StringBuilder();

        for (StackTraceElement temp : stackTraceElements) {
            stringBuilder.append(temp.toString() + "\n");
        }

        return stringBuilder.toString();
    }
}

```

##### 3.注册Xposed自定义模块

在main下新建```assets```资源文件，并新建名为```xposed_init```文件，在这个文件里面添加 ```Xposed模块的包名+类名```。
如：

```
com.demon.xposed_hook.XposedHook
```

在```AndroidManifest.xml```中添加：

```xml
    <meta-data
            android:name="xposedmodule"
            android:value="true" />

        <!--模块说明，一般为模块的功能描述-->
        <meta-data
            android:name="xposeddescription"
            android:value="这个模块是用来检测用户隐私合规的，在用户未授权同意前，调用接口获取信息属于违规" />

        <!--模块兼容版本-->
        <meta-data
            android:name="xposedminversion"
            android:value="54" />
```

#### 3.将Xposed自定义模块添加到Xposed虚拟机中

编译运行项目，安装到手机。

启动VirtualXposed--->点击菜单：

1. 添加应用--->找到Xposed模块应用--->选中后安装
2. 模块管理--->勾选--->重启VirtualXposed


#### 4.将需要检测的App添加到Xposed虚拟机中

启动VirtualXposed--->点击菜单-->添加应用--->找到你需要检测的App--->选中后安装

#### 5. 在VirtualXposed中开始检测

在VirtualXposed中先后启动：Xposed Installer--->Xposed自定义模块--->需要检测的App

#### 6. 将Xposed Installer中的日志导出，并分析
已我们的App的日志为例，分析日志中App启动会获取MAC的第三方有：

##### 1.bugly

```
04-26 15:54:07.828 I/Xposed  (11539): com.tencent.bugly.crashreport.common.info.b.d(BUGLY:3)
04-26 15:54:07.828 I/Xposed  (11539): com.tencent.bugly.crashreport.common.info.a.k(BUGLY:4)
04-26 15:54:07.828 I/Xposed  (11539): com.tencent.bugly.proguard.a.a(BUGLY:179)
04-26 15:54:07.828 I/Xposed  (11539): com.tencent.bugly.crashreport.biz.a.b(BUGLY:35)
04-26 15:54:07.828 I/Xposed  (11539): com.tencent.bugly.crashreport.biz.a$a.run(BUGLY:6)

```

##### 2.讯飞

```
04-26 16:14:43.378 I/Xposed  (21437): com.iflytek.cloud.thirdparty.ad.d(Unknown Source:12)
04-26 16:14:43.378 I/Xposed  (21437): com.iflytek.cloud.thirdparty.ad.g(Unknown Source:238)
04-26 16:14:43.378 I/Xposed  (21437): com.iflytek.cloud.thirdparty.ad.a(Unknown Source:11)
04-26 16:14:43.378 I/Xposed  (21437): com.iflytek.cloud.thirdparty.at.a(Unknown Source:47)
04-26 16:14:43.378 I/Xposed  (21437): com.iflytek.cloud.thirdparty.x.a(Unknown Source:3)
04-26 16:14:43.378 I/Xposed  (21437): com.iflytek.cloud.thirdparty.y.a(Unknown Source:8)
04-26 16:14:43.378 I/Xposed  (21437): com.iflytek.cloud.SpeechUtility.b(Unknown Source:40)
04-26 16:14:43.378 I/Xposed  (21437): com.iflytek.cloud.SpeechUtility.<init>(Unknown Source:61)
04-26 16:14:43.378 I/Xposed  (21437): com.iflytek.cloud.SpeechUtility.createUtility(Unknown Source:48)
```

##### 3.友盟

```
04-26 16:14:43.038 I/Xposed  (21437): com.umeng.message.common.UmengMessageDeviceConfig.getMac(UmengMessageDeviceConfig.java:11)
04-26 16:14:43.038 I/Xposed  (21437): com.umeng.message.common.b.a(Header.java:19)
04-26 16:14:43.038 I/Xposed  (21437): com.umeng.message.common.b.c(Header.java:8)
04-26 16:14:43.038 I/Xposed  (21437): com.umeng.message.UTrack.updateHeader(UTrack.java:11)
04-26 16:14:43.038 I/Xposed  (21437): com.umeng.message.MessageSharedPrefs$1.run(MessageSharedPrefs.java:1)
```

将这些第三方库都放到同意隐私权限后再初始化，再次按照上面的方法使用Xposed检测。
直到日志中App启动后已经没有任何地方调用获取敏感信息的方法即可。
