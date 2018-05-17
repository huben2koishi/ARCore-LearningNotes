# CameraPermissionHelper
这个类用于帮助请求相机权限等问题

## 公共静态方法
### public static boolean hasCameraPermission(Activity activity)

检查app拥有必要的权限，如果有，返回ture

### public static void requestCameraPermission(Activity activity)

如果app没有必要的权限，请求这个权限

### public static boolean shouldShowRequestPermissionRationale(Activity activity)

如果应用之前请求过此权限但用户拒绝了请求，返回 true，如果用户拒绝授予权限且点击了“不再提醒”，返回ture

### public static void launchPermissionSettings(Activity activity)

启动app的设置界面以授予权限

## 调用
在HelloArActivity.onResume()中，用hasCameraPermission(this)判断应用是否有使用相机的权限，如果没有，调用requestCameraPermission(this)请求这个权限。
``` java
if (!CameraPermissionHelper.hasCameraPermission(this)) {
  CameraPermissionHelper.requestCameraPermission(this);
  return;
}
```

在HelloArActivity.onRequestPermissionsResult()中，用hasCameraPermission(this)判断应用是否有使用相机的权限，如果没有，显示一个Toast消息，再调用shouldShowRequestPermissionRationale(this)判断用户是否拒绝授予权限且点击了“不再提醒”，如果是，调用launchPermissionSettings(this)打开应用设置以授予权限。
``` java
if (!CameraPermissionHelper.hasCameraPermission(this)) {
  Toast.makeText(this, "Camera permission is needed to run this application", Toast.LENGTH_LONG).show();
  if (!CameraPermissionHelper.shouldShowRequestPermissionRationale(this)) {
    CameraPermissionHelper.launchPermissionSettings(this);
  }
  finish();
}
```