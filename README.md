实践过程 CSDN 有写
https://blog.csdn.net/sinat_35093406/article/details/87938040

# react-native-hot-deployment

## Getting started

`$ npm install react-native-hot-deployment --save`

### Mostly automatic installation

`$ react-native link react-native-hot-deployment`

### Manual installation

#### Android

1. Open up `android/app/src/main/java/[...]/MainApplication.java`

- Add `import com.xiaomo.HotDeploymentPackage;;` to the imports at the top of the file
- Add `new HotDeploymentPackage()` to the list returned by the `getPackages()` method

2. Append the following lines to `android/settings.gradle`:

   ```
   include ':react-native-hot-deployment'
   project(':react-native-hot-deployment').projectDir = new File(rootProject.projectDir, 	'../node_modules/react-native-hot-deployment/android')
   ```

3. Insert the following lines inside the dependencies block in `android/app/build.gradle`:
   ```
   compile project(':react-native-hot-deployment')
   ```
4. add `MainApplication`

   ```
   import android.content.SharedPreferences;
   import android.content.pm.PackageInfo;
   import android.content.pm.PackageManager;
   import java.io.File;
   private SharedPreferences sharedPreferences;

   private String newVersion = "";
   private final ReactNativeHost mReactNativeHost = new ReactNativeHost(this) {
     @Override
     protected String getJSBundleFile() {
       PackageManager pm = getApplicationContext().getPackageManager();
       PackageInfo pi;
       try {
           pi = pm.getPackageInfo(getApplicationContext().getPackageName(), 0);
           newVersion = pi.versionName;
       } catch (PackageManager.NameNotFoundException e) {
           e.printStackTrace();
       }
       sharedPreferences=getSharedPreferences("appVersion.xml",MODE_PRIVATE);
       String oldVersion=sharedPreferences.getString("version","0.0.0");
       String jsBundleFile =  getFilesDir().getAbsolutePath()+"/index.android.bundle";
       File file = new File(jsBundleFile);
       int oldVersionBack=Integer.parseInt(oldVersion.replaceAll("\\.",""));
       int newVersionBack=Integer.parseInt(newVersion.replaceAll("\\.",""));
       if((oldVersion.equals("0.0.0")|| oldVersionBack>=newVersionBack)&& file != null && file.exists()) {
         return jsBundleFile;
       } else {
         return super.getJSBundleFile();
       }
     }
   }
   ```

## Usage

```javascript
// TODO: What to do with the module?
import { NativeModules } from "react-native";
const { HotPushVersion, Download, DownloadApk, RestartApp } = NativeModules;

//下载安装包，下载完成后自动跳转安装界面
DownloadApk.downloading(url, "描述");

//下载文件压缩包，地址为自己服务器上部署的地址，压缩包命名应为bundle.zip，里面包含index.android.bundle和资源文件(drawable\drawable-****)
Download.downloading(url);

//下载进度监听，下载完成后可调用RestartApp.Restart()重启app
this.listener = DeviceEventEmitter.addListener('downloadZipStatus', (e) => {
	if (e && e.status === "success") {
    //下载成功
    //需要传入最新版本号，目前插件只支持版本号格式为x.x.x
    HotPushVersion.updataVersion(version);
		RestartApp.Restart()
  } else if(e && e.status === "error") {
    //下载失败
  }else if(e && e.status === "decompressionError") {
    //解压bundle.zip文件失败
  }else if (e && e.status) {
		//下载进度
		console.log("下载进度====>" + e.status + "%")
	} else {
		//失败
	}
})

//重新启动app
RestartApp.Restart()


//一般来说，后端需要提供一个接口用来判断是否需要更新和以什么方式来更新。
//下载安装包的方式只需要调用DownloadApk.downloading(url, "描述");
//热更新方式则须使用以下命令打包资源文件。
react-native bundle --platform android --dev false --entry-file index.js  --bundle-output bundle_zip/index.android.bundle  --assets-dest bundle_zip

//然后将资源文件放置服务器以供下载Download.downloading(url);下载完成后调用RestartApp.Restart()重启APP即可完成热更新。当然打包资源文件前需要在package更改版本号。(这里我们需要改变一下版本号的获取方式，在app的build.gradle最外层加上如下行代码，这样就只需修改package.json里面的版本号。)

import com.android.build.OutputFile
import groovy.json.JsonSlurper
def getAppVersion(){
    def inputFile = new File("../package.json")
    def packageJson = new JsonSlurper().parseText(inputFile.text)
    return packageJson["version"]
}
def appVersion = getAppVersion()
android{defaultconfig{versionName appVersion}}


//会android原生的同学可以把插件拉取下来根据需求自行更改
```
