# ARCore提供的信息

## session和config
``` java
session = new Session(this);
Config config = new Config(session);
    if (!session.isSupported(config)) {
        showSnackbarMessage("This device does not support AR", true);
    }
session.configure(config);
```
- Session：ARCore的管理类，它非常重要。ARCore的打开，关闭，视频帧的获取等都是通过它来管理的

- Config：存放一些配置信息，如平面的查找模式，光照模式等信息都是记录在该类中

- isSupported()：该方法主要是对 SDK的版本及机型做检测

## frame和camera
通过上面的session调用update()方法更新ARCore系统的状态。这包括：接收新的摄像机frame，更新设备的位置，更新跟踪anchor的位置，更新检测到的plane等。
再通过frame调用getCamera()方法获取camera对象，此Camera实例是长期存在的，因此无论此方法被调用的frame对象如何，都会返回相同的实例
``` java
Frame frame = session.update();
Camera camera = frame.getCamera();
```
- frame：用于测试点击、绘制背景、获取光照信息以及获取识别到的特征点
- camera：用于获取当前camera相对世界坐标系的投影矩阵和视图矩阵、摄像头姿态信息

## 投影(Project)矩阵和视图(View)矩阵
``` java
float[] projmtx = new float[16];
camera.getProjectionMatrix(projmtx, 0, 0.1f, 100.0f);

float[] viewmtx = new float[16];
camera.getViewMatrix(viewmtx, 0);
```
通过上面的代码获取投影矩阵和视图矩阵，这两个矩阵决定了虚拟世界里的哪些物体能够被看见

## 点云(pointCloud)
``` java
PointCloud pointCloud = frame.acquirePointCloud();
```
通过上面的代码获取识别到的特征点

## 锚点(anchor)
``` java
MotionEvent tap = queuedSingleTaps.poll();
if (tap != null && camera.getTrackingState() == TrackingState.TRACKING) {
    for (HitResult hit : frame.hitTest(tap)) {
        // 检查是否点击到了平面
        Trackable trackable = hit.getTrackable();
        // 如果点击到一个平面或导向点，则创造一个Anchor
        if ((trackable instanceof Plane && ((Plane) trackable).isPoseInPolygon(hit.getHitPose())) || (trackable instanceof Point == OrientationMode.ESTIMATED_SURFACE_NORMAL)) {
            if (anchors.size() >= 20) {
                anchors.get(0).detach();
                anchors.remove(0);
            }
            anchors.add(hit.createAnchor());
            break;
        }
    }
}
```
先查看是否有点击事件，如果是，且图像处理于跟踪状态，就对其进行命中检测，看是否可以找到一个平面，如果找到就创建一个锚点`hit.createAnchor()`并将其与该平台绑定起来`anchors.add(hit.createAnchor())`

## 物体的姿态(pose)
``` java
anchor.getPose().toMatrix(anchorMatrix, 0);
```
通过上面的代码，就可以将物体的姿态信息转成矩阵，包含姿态、位置信息

## 光照信息
``` java
final float lightIntensity = frame.getLightEstimate().getPixelIntensity();
```
getLightEstimate()返回当前的环境光线估计值，getPixelIntensity()
返回当前摄像机视图的像素强度。得到的lightIntensity用于绘制物品及其阴影

# 信息的使用

## 绘制背景
``` java
backgroundRenderer.draw(frame);
```
调用BackgroundRenderer的draw()方法，即可将摄像头捕获的图像数据作为背景绘制出来

|参数||
|---|---|
|frame|session.update()获取的摄像机frame|

## 绘制点云
``` java
this.pointCloud.update(pointCloud);
this.pointCloud.draw(viewmtx, projmtx);
```
调用PointCloudRenderer的update()方法更新识别到的特征点，再调用draw()方法即可绘制ARCore的点云

|参数||
|---|---|
|pointCloud|识别到的特征点|
|viewmtx|视图矩阵|
|projmtx|投影矩阵|

## 绘制平面
``` java
planeRenderer.drawPlanes(session.getAllTrackables(Plane.class), camera.getDisplayOrientedPose(), projmtx);
```
调用PointlaneRenderer的drawPlanes()方法即可绘制ARCore的识别出的所有平面

|参数||
|---|---|
|session.getAllTrackables(Plane.class)|识别出的所有平面|
|camera.getDisplayOrientedPose()|设备在世界坐标系中的pose|
|projmtx|投影矩阵|

## 绘制物品及阴影
``` java
virtualObject.updateModelMatrix(anchorMatrix, scaleFactor);
virtualObjectShadow.updateModelMatrix(anchorMatrix, scaleFactor);
virtualObject.draw(viewmtx, projmtx, lightIntensity);
virtualObjectShadow.draw(viewmtx, projmtx, lightIntensity);
```
调用ObjectRenderer的updateModelMatrix()方法更新对象模型矩阵并应用缩放，再调用draw()方法即可绘制放置的虚拟物体和它的阴影

|参数||
|---|---|
|anchorMatrix|物体的姿态信息|
|scaleFactor|缩放比例|
|viewmtx|视图矩阵|
|projmtx|投影矩阵|
|lightIntensity|光照信息|