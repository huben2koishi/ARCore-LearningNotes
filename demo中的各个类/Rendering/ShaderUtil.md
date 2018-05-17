# ShaderUtil
这个类用于提供绘制用到的工具

# 公共静态方法
### public static int loadGLShader(String tag, Context context, int type, String filename)

将原始文本文件(保存为资源)转换为OpenGL ES着色程序

### public static void checkGLError(String tag, String label)

检查OpenGL ES中是否有错误，如果有的话，是什么错误

# 私有静态方法
### private static String readRawTextFileFromAssets(Context context, String filename)

将原始文本文件转换为字符串：从原始资源中通过流读取字符串