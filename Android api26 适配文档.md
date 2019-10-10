#### 人社app api26 适配文档
[TOC]
#### 1.  6.0适配(动态权限)
**6.0适配主要是运行时权限适配**

- 危险权限列表(Dangerous Permission)

  ![img](https://img-blog.csdn.net/20160724115427239)

**目前人社app用到的已知权限**(以下权限已经适配)：

```xml
 Manifest.permission.READ_CONTACTS
 Manifest.permission.WRITE_EXTERNAL_STORAGE
 Manifest.permission.CAMERA
 Manifest.permission.CALL_PHONE
 Manifest.permission.ACCESS_FINE_LOCATION
```

人社动态权限适配再LauncherActivity中

```java
TedPermission.with(LauncherActivity.this)
             .setPermissionListener(new PermissionListener() {
                  @Override
                 public void onPermissionGranted() {
                    FastPref.putTestBaseUrl(urlTv.getText().toString());
                    getCertAndUrl();
                 }

                 @Override
                 public void onPermissionDenied(List<String> list) {
                     ToastUtils.show("请在设置中开启相应权限,否则app无法正常运行");
                                    finish();
                      }
                 })
                 .setDeniedMessage("请在设置中开启相应权限,否则app无法正常运行")
                 .setPermissions(Manifest.permission.READ_CONTACTS,
                                    Manifest.permission.WRITE_EXTERNAL_STORAGE,
                                    Manifest.permission.CAMERA,
                                    Manifest.permission.CALL_PHONE,
                                    Manifest.permission.ACCESS_FINE_LOCATION)
                 .check();
```

#### 2.  7.0适配(fileprovider)

对于面向 Android 7.0 的应用，Android 框架执行的 StrictMode API 政策禁止在您的应用外部公开 file:// URI。如果一项包含文件 URI 的 intent 离开您的应用，则应用出现故障，并出现 FileUriExposedException 异常

**使用FileProvider** 

FileProvider使用大概分为以下几个步骤：

- manifest中申明FileProvider

  ```xml
   <provider
            android:name="android.support.v4.content.FileProvider"
            android:authorities="cn.com.dareway.${PKG}.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true"
            tools:replace="android:authorities">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths"
                tools:replace="android:resource" />
        </provider>
  ```

- res/xml中定义对外暴露的文件夹路径

  ```xml
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-path name="my_images" path="/"/>
</paths>
  ```

- 生成content://类型的Uri

  ```java
  Uri uri = null;
   if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
      uri = FileProvider.getUriForFile(this, getPackageName()+".fileprovider", file);
   } else {
      uri = Uri.fromFile(file);
   }
  ```

- 给Uri授予临时权限

  ```java
  intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION
               | Intent.FLAG_GRANT_WRITE_URI_PERMISSION);
  ```

- 使用Intent传递Uri

  **人社中涉及到的主要有拍照(需测试),裁剪图片(需测试),更新安装应用(已适配)** 

  拍照及裁剪图片可参照UploadSiPhotoActivity


  #### 3.  8.0适配

 ##### 1.通知栏适配(参考UpdateService  也可参考 https://blog.csdn.net/guolin_blog/article/details/79854070)
```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O && Flavor.current().isHighAPi()) {
            notificationManager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
            NotificationChannel mChannel = new NotificationChannel("update", "应用更新", NotificationManager.IMPORTANCE_HIGH);
            notificationManager.createNotificationChannel(mChannel);
            notiBuilder = new Notification.Builder(this, "update")
                    .setContentTitle("正在更新...") //设置通知标题
                    .setSmallIcon(R.mipmap.ic_launcher)
                    .setLargeIcon(BitmapFactory.decodeResource(BaseApplication.getApplication().getResources(), R.mipmap.ic_launcher)) //设置通知的大图标
                    .setAutoCancel(false)//设置通知被点击一次是否自动取消
                    .setContentText("下载进度:" + "0%")
                    .setProgress(100, 0, false);
            notification = notiBuilder.build();
        } else {
            notificationManager = (NotificationManager) BaseApplication.getApplication().getSystemService(Context.NOTIFICATION_SERVICE);
            builder = new NotificationCompat.Builder(BaseApplication.getApplication());
            builder.setContentTitle("正在更新...") //设置通知标题
                    .setSmallIcon(R.mipmap.ic_launcher)
                    .setLargeIcon(BitmapFactory.decodeResource(BaseApplication.getApplication().getResources(), R.mipmap.ic_launcher)) //设置通知的大图标
                    .setDefaults(Notification.DEFAULT_LIGHTS) //设置通知的提醒方式： 呼吸灯
                    .setPriority(NotificationCompat.PRIORITY_MAX) //设置通知的优先级：最大
                    .setAutoCancel(false)//设置通知被点击一次是否自动取消
                    .setContentText("下载进度:" + "0%")
                    .setProgress(100, 0, false);
            notification = builder.build();//构建通知对象
        }
```
##### 2. 悬浮窗适配(暂未用到)

使用 SYSTEM_ALERT_WINDOW 权限的应用无法再使用以下窗口类型来在其他应用和系统窗口上方显示提醒窗口：

- TYPE_PHONE

- TYPE_PRIORITY_PHONE

- TYPE_SYSTEM_ALERT

- TYPE_SYSTEM_OVERLAY

- TYPE_SYSTEM_ERROR

相反，应用必须使用名为 `TYPE_APPLICATION_OVERLAY` 的新窗口类型。

```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
    mWindowParams.type = WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY
}else {
    mWindowParams.type = WindowManager.LayoutParams.TYPE_SYSTEM_ALERT
}
```

需要有权限

```xml
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
<uses-permission android:name="android.permission.SYSTEM_OVERLAY_WINDOW" />
```

##### 3. 安装apk

需在`AndroidManifest`文件中添加安装未知来源应用的权限:
```xml
<uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES"/>
```

##### 4. 权限(暂未用到)

在Android 8.0之前，如果应用在运行时请求某个权限并且被授予，系统会错误地将属于同一权限组并且在清单中注册的其他权限也一并授予该应用。对于Android 8.0的应用，此行为已被纠正。系统只会授予应用明确请求的权限。然而，一旦用户为应用授予某个权限，则所有后续对该权限组中权限的请求都将被自动批准，而不会提示用户

比如：8.0之前你申请读外部存储的权限READ_EXTERNAL_STORAGE，你会自动被赋予写外部存储的权限WRITE_EXTERNAL_STORAGE，因为他们属于同一组(android.permission-group.STORAGE)权限，但是现在8.0不一样了，读就是读，写就是写，不能混为一谈。不过你授予了读之后，虽然下次还是要申请写，但是在申请的时候，申请会直接通过，不会让用户再授权一次了

##### 5. 应用图标(暂未用到)

参考 https://blog.csdn.net/guolin_blog/article/details/79417483

#### 人社app适配

1. app下build.gradle添加 targetSdkVersion 26
```
yingkou {
      applicationId "cn.com.dareway.mhsc_yingkou"
      targetSdkVersion 26
      manifestPlaceholders = [CHANNEL_NAME: "mhsc_yingkou", PKG:...]
      }
```

2. Flavor类中设置最后highAPi为true
```java
 yingKou("yingkou","2108","营口","Y",true)
```

3. 具体功能测试,哪里出现bug适配哪里



