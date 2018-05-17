# HelloArActivity
demo的MainActivity

## 方法
- onCreate(Bundle savedInstanceState)
指定surfaceView，创建displayRotationHelper，设置点击监听器，设置渲染器

- onResume()
检测相机权限，初始化session并配置config，初始化surfaceView，displayRotationHelper

- onPause()
暂停session，surfaceView，displayRotationHelper

- onRequestPermissionsResult(int requestCode, String[] permissions, int[] results)
运行时权限设置

- onWindowFocusChanged(boolean hasFocus)
Android全屏功能

- onSurfaceCreated(GL10 gl, EGLConfig config)
- onSurfaceChanged(GL10 gl, int width, int height)
- onDrawFrame(GL10 gl)
详见[draw.md](https://github.com/huben2koishi/ARCore-LearningNotes/blob/master/draw.md)