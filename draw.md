## ARCore Demo中图形绘制

### GLSurfaceView

GLSurfaceView是Android应用程序中实现OpenGl画图的重要组成部分。GLSurfaceView中封装了一个Surface。而android平台下关于图像的现实，差不多都是由Surface来实现的。
在HelloArActivity.onCreate()中配置GLSurfaceView。相关代码如下：
``` java
surfaceView = findViewById(R.id.surfaceview);
...
surfaceView.setPreserveEGLContextOnPause(true);//在pause状态下，保留EGL上下文
surfaceView.setEGLContextClientVersion(2);//OpenGL ES 版本选择 2.0 版本
surfaceView.setEGLConfigChooser(8, 8, 8, 8, 16, 0);//绘制表面选择RGBA分别为8位，深度16位，模板0位的配置
surfaceView.setRenderer(this);//设置自身为渲染器，即处理逻辑在HelloArActivity这个类里实现
surfaceView.setRenderMode(GLSurfaceView.RENDERMODE_CONTINUOUSLY);//渲染模式设为持续渲染，即一帧渲染完，马上开始下一帧的渲染
```

### Renderer

有了GLSurfaceView之后，就相当于我们有了画图的纸。现在我们所需要做的就是如何在这张纸上画图。所以我们需要一支笔。
Renderer是GLSurfaceView的内部静态接口，它就相当于在此GLSurfaceView上作画的笔。我们通过实现这个接口来“作画”。最后通过GLSurfaceView的setRenderer(GLSurfaceView.Renderer renderer)方法，就可以将纸笔关联起来了。

demo中的几个实现的Renderer：
- backgroundRenderer    用于绘制摄像头采集到的数据
- virtualObject         用于绘制Android小机器人
- virtualObjectShadow   用于给Android机器人绘制阴影
- planeRenderer         用于绘制SDK识别出来的平面
- pointCloud            用于绘制SDK识别出来的点云

### 绘制的实现
要实现在GLSurfaceView上绘制内容，需实现GLSurfaceView.Renderer接口。此接口定义如下：
``` java
public interface Renderer {
    void onSurfaceCreated(GL10 gl, EGLConfig config);

    void onSurfaceChanged(GL10 gl, int width, int height);

    void onDrawFrame(GL10 gl);
}
```
- onSurfaceCreated(GL10 gl, EGLConfig config)
    这个方法在在绘制表面创建或重新创建的时候被调用。在这个函数的实现中做一些初始化的工作。第一个参数非常重要。如果说Renderer是画笔的话，那么这个gl参数，就可以说是我们的手了。如何操作这支画笔，都是它说了算。绝大部分时候都是通过gl来指挥Renderer画图的。
- void onSurfaceChanged(GL10 gl, int width, int height);
    这个方法在在绘制表面发生变化的时候被调用。此时外部可能改变了控件的大小，因此我们需要在这个调用里更新我们的视口信息，以便绘制的时候能准确绘制到屏幕中来。值得注意的是，在Surface刚创建的时候，它的size其实是0，也就是说在画第一次图之前它也会被调用一次的。当然对于很多时候，Surface的大小是不会改变的，那么此函数就只在创建之初被调用一次而已。
- void onDrawFrame(GL10 gl);
    这个方法在绘制的时候调用。有两种绘制模式：
    1. RENDERMODE_CONTINUOUSLY每绘制一次，就会调用一次，即每一帧触发一次，这种模式很明显是用来画动画的。
    2. RENDERMODE_WHEN_DIRTY只有在需要重画的时候才画下一幅。这种模式就比较节约CPU和GPU一些，适合用来画不经常需要刷新的情况。
    demo中用的显然是第一种。

### 绘制逻辑
- 首先，初始化：
``` java
public void onSurfaceCreated(GL10 gl, EGLConfig config) {

    GLES20.glClearColor(0.1f, 0.1f, 0.1f, 1.0f);

    backgroundRenderer.createOnGlThread(this);

    try {
        virtualObject.createOnGlThread(this, "andy.obj", "andy.png");
        virtualObject.setMaterialProperties(0.0f, 3.5f, 1.0f, 6.0f);

        virtualObjectShadow.createOnGlThread( this, "andy_shadow.obj", "andy_shadow.png");
        virtualObjectShadow.setBlendMode(BlendMode.Shadow);
        virtualObjectShadow.setMaterialProperties(1.0f, 0.0f, 0.0f, 1.0f);
    } catch (IOException e) {
        Log.e(TAG, "Failed to read obj file");
    }
    try {
        planeRenderer.createOnGlThread(this, "trigrid.png");
    } catch (IOException e) {
        Log.e(TAG, "Failed to read plane texture");
    }

    pointCloud.createOnGlThread(this);
}
```
- 然后是配置绘制表面的大小，把绘制表面的size信息通知给ARCore
``` java
public void onSurfaceChanged(GL10 gl, int width, int height) {
    displayRotationHelper.onSurfaceChanged(width, height);
    GLES20.glViewport(0, 0, width, height);
}
```
- 最后，就是核心的绘制部分，这部分较长，仅保留核心部分：
``` java
public void onDrawFrame(GL10 gl) {
    GLES20.glClear(GLES20.GL_COLOR_BUFFER_BIT | GLES20.GL_DEPTH_BUFFER_BIT);

    if (session == null) {
        return;
    }

    try {
        backgroundRenderer.draw(frame);

        if (camera.getTrackingState() == TrackingState.PAUSED) {
            return;
        }
        PointCloud pointCloud = frame.acquirePointCloud();
        this.pointCloud.update(pointCloud);
        this.pointCloud.draw(viewmtx, projmtx);

        planeRenderer.drawPlanes(session.getAllTrackables(Plane.class), camera.getDisplayOrientedPose(), projmtx);

        for (Anchor anchor : anchors) {
            if (anchor.getTrackingState() != TrackingState.TRACKING) {
                continue;
            }

            anchor.getPose().toMatrix(anchorMatrix, 0);

            virtualObject.updateModelMatrix(anchorMatrix, scaleFactor);
            virtualObjectShadow.updateModelMatrix(anchorMatrix, scaleFactor);
            virtualObject.draw(viewmtx, projmtx, lightIntensity);
            virtualObjectShadow.draw(viewmtx, projmtx, lightIntensity);
        }

    } catch (Throwable t) {
        Log.e(TAG, "Exception on the OpenGL thread", t);
    }
}
```

绘制中:

- backgroundRenderer绘制摄像头拍到的内容
- vrtualObject、virtualObjectShadow绘制虚拟物体和它的阴影
- planeRenderer绘制ARCore识别出来的平面
- pointCloud绘制ARCore识别出来的特征点云

绘制相关的方法都是draw()或drawXXX()，而具体的实现逻辑都封装在了相应的xxxRenderer类中。
在绘制之前，这些负责绘制的对象都需要我们提供一些信息：

- 绘制的物体的位置
- 绘制的物体视图(View)矩阵，投影(Project)矩阵
- 物体的姿态(位置和朝向)
- 物体的光照信息

而这些信息都是通过ARCore来取得的