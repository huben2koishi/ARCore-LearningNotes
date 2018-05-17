# DisplayRotationHelper
这个类用于帮助跟踪显示旋转

## 公共方法
### public void onResume()

注册显示监听器。在Activity.onResume()中调用

### public void onPause()

取消注册显示监听器。在Activity.onPause()中调用

### public void onSurfaceChanged(int width, int height)

记录表面尺寸的变化。 将在以后由updateSessionIfNeeded(Session)使用。
从onSurfaceChanged(GL10，int，int)中调用

### public void updateSessionIfNeeded(Session session)

如果通过onSurfaceChanged(int，int)调用或通过onDisplayChanged(int)系统回调发布更改，则更新session
在每次调用Session.update()之前，都应该明确调用该函数