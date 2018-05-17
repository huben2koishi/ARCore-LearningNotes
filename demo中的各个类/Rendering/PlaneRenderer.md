# PlaneRenderer
这个类用于绘制ARCore识别出来的平面

## public 方法
### public void createOnGlThread(Context context, String gridDistanceTextureName)

初始化平面渲染器,HelloArActivity.onSurfaceCreated()中调用
``` java
planeRenderer.createOnGlThread(this, "models/trigrid.png");
```

### public void drawPlanes(Collection<Plane### allPlanes, Pose cameraPose, float[] cameraPerspective)

绘制识别出的所有平面,HelloArActivity.onDrawFrame()中调用
``` java
Camera camera = frame.getCamera();
float[] projmtx = new float[16];
camera.getProjectionMatrix(projmtx, 0, 0.1f, 100.0f);
planeRenderer.drawPlanes(session.getAllTrackables(Plane.class), camera.getDisplayOrientedPose(), projmtx);
```


## private 方法
### private void updatePlaneParameters(float[] planeMatrix, float extentX, float extentZ, FloatBuffer boundary)

更新平面模型变换矩阵和范围,drawPlanes()中调用

### private void draw(float[] cameraView, float[] cameraPerspective)

绘制平面,drawPlanes()中调用