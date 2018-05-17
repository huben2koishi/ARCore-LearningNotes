# ObjectRenderer
这个类用于绘制从OpenGL中的OBJ文件加载的对象

## public 方法
### public void createOnGlThread(Context context, String objAssetName, String diffuseTextureAssetName)

初始化物品渲染器,HelloArActivity.onSurfaceCreated()中调用
``` java
virtualObject.createOnGlThread(this, "models/andy.obj", "models/andy.png");
virtualObjectShadow.createOnGlThread(this, "models/andy_shadow.obj", "models/andy_shadow.png");
```

### setBlendMode(BlendMode xuanMode)

设置混合模式,HelloArActivity.onSurfaceCreated()中调用
``` java
virtualObjectShadow.setBlendMode(BlendMode.Shadow);
```

### public void updateModelMatrix(float[] modelMatrix, float scaleFactor)

更新对象模型矩阵并应用缩放,HelloArActivity.onDrawFrame()中调用
``` java
float scaleFactor = 1.0f;
anchor.getPose().toMatrix(anchorMatrix, 0);
virtualObject.updateModelMatrix(anchorMatrix, scaleFactor);
virtualObjectShadow.updateModelMatrix(anchorMatrix, scaleFactor);
```

### public void setMaterialProperties(float ambient, float diffuse, float specular, float specularPower)

设置材质信息,HelloArActivity.onSurfaceCreated()中调用
``` java
virtualObject.setMaterialProperties(0.0f, 2.0f, 0.5f, 6.0f);
virtualObjectShadow.setMaterialProperties(1.0f, 0.0f, 0.0f, 1.0f);
```

### public void draw(float[] cameraView, float[] cameraPerspective, float[] colorCorrectionRgba)

绘制放置的虚拟物体和它的阴影,HelloArActivity.onDrawFrame()中调用
``` java
float[] projmtx = new float[16];
camera.getProjectionMatrix(projmtx, 0, 0.1f, 100.0f);
float[] viewmtx = new float[16];
camera.getViewMatrix(viewmtx, 0);
final float[] colorCorrectionRgba = new float[4];
frame.getLightEstimate().getColorCorrection(colorCorrectionRgba, 0);

virtualObject.draw(viewmtx, projmtx, colorCorrectionRgba;
virtualObjectShadow.draw(viewmtx, projmtx, colorCorrectionRgba);
```