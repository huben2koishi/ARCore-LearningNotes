# BackgroundRenderer
这个类用于绘制摄像头采集到的数据

## public 方法
### public void createOnGlThread(Context context)

初始化背景渲染器,HelloArActivity.onSurfaceCreated()中调用
``` java
backgroundRenderer.createOnGlThread(this);
```
### public void draw(Frame frame)

绘制背景,即摄像头捕获的图像数据,HelloArActivity.onDrawFrame()中调用
``` java
backgroundRenderer.draw(frame);
```

### public int getTextureId()

设置允许GPU访问相机图像的OpenGL纹理名称（id）,HelloArActivity.onDrawFrame()中调用
``` java
session.setCameraTextureName(backgroundRenderer.getTextureId());
```