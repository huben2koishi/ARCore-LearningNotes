# TapHelper
这个类用于帮助使用GestureDetector检点击，并在UI线程和渲染线程之间传递点击

# 公共方法
### public MotionEvent poll()

取出BlockingQueue里排在首位的tap,如果取不到返回null