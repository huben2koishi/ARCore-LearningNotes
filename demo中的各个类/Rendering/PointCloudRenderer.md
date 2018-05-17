# PointCloudRenderer
这个类用于绘制ARCore识别出来的点云

## public 方法
> createOnGlThread(Context context)

初始化点云渲染器,HelloArActivity.onSurfaceCreated()中调用
``` java
pointCloud.createOnGlThread(this);
```

> update(PointCloud cloud)

更新点云，重复的将会忽略,HelloArActivity.onDrawFrame()中调用
``` java
PointCloud pointCloud = frame.acquirePointCloud();
this.pointCloud.update(pointCloud);
```

> draw(float[] cameraView, float[] cameraPerspective)

绘制点云,HelloArActivity.onDrawFrame()中调用
``` java
float[] projmtx = new float[16];
camera.getProjectionMatrix(projmtx, 0, 0.1f, 100.0f);
float[] viewmtx = new float[16];
camera.getViewMatrix(viewmtx, 0);
this.pointCloud.draw(viewmtx, projmtx);
```
