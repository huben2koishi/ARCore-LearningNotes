# ARCore的核心类

## [Session](https://github.com/huben2koishi/ARCore-LearningNotes/wiki/Session)
Session是ARCore中重要的功能，它管理AR系统的状态，处理Session生命周期，是ARCore API的主要入口。

在开始使用ARCore API的时候，通过设置的Config来检查当前设备是否支持ARCore。在Activity中对应的生命周期方法中需要处理Session的生命周期，这样AR系统会根据需要开始和暂停相机帧的采集，初始化和释放相关的资源。

在AR系统在每一帧画面的渲染过程中，我们从Session中获取当前相机反馈的Frame，根据需要保存、获取和删除Anchor，获取系统检测到的所有Plane，设置纵横缩放比，获取投影矩阵（渲染3d效果）等用于渲染相关工作。

## [Config](https://github.com/huben2koishi/ARCore-LearningNotes/wiki/Config)
在当前市面上，并不是所有的Android设备都支持ARCore。那么这时候我们就要用到Config，它保存了用于配置Session相关设置。

我们可以针对是否开启平面检测和光线感知，获取帧时没有新图片是否阻塞等几个选项创建Config。默认Config开启平面检测和光线感知，获取帧时没有新图片阻塞。 在使用ARCore之前，使用创建的Config，检查当前设备是否支持ARCore，是一个比较好的做法

## [Frame](https://github.com/huben2koishi/ARCore-LearningNotes/wiki/Frame)
Frame最直观的理解是照相机获取的一帧图像，Demo背景渲染的画面就来自摄像头获取图像帧。

在ARCore中，Frame还包含更丰富的内容，还提供了某一个时刻AR的状态。这些状态包括：当前帧中环境的光线，如在绘制内容的时候根据光线控制物体绘制的颜色，使得更逼真；当前Frame中检测到的特征点云和它的Pose用来绘制点云；当前Frame中包含的Anchor和检测到的Plane集合用于绘制内容和平面；手机设备当前的Pose、帧获取的时间戳、AR跟踪状态和摄像头的视图矩阵等。

该类通过调用update()方法，获取状态信息并更新AR系统

## [HitResult](https://github.com/huben2koishi/ARCore-LearningNotes/wiki/HitResult)
点击平面的时候，从设备点击开始手机朝向方向发出一条射线，和被检测平面的是否有交集。如果就在交集处绘制Android机器人。

抽象的概括，HitResult就是射线和真实世界几何体的交集。我们可以中获取当前设备到有交集几何体的距离，交集的Pose。

## [Point](https://github.com/huben2koishi/ARCore-LearningNotes/wiki/Point)
它代表ARCore正在跟踪的空间点。它是创建锚点（调用createAnchor方法）时，或者进行命中检测（调用hitTest方法）时返回的结果

## [PointCloud](https://github.com/huben2koishi/ARCore-LearningNotes/wiki/PointCloud)
点云包含了被观察到的3d点和置信度值的集合，还有它被ARCore检测到的时间戳

## [Plane](https://github.com/huben2koishi/ARCore-LearningNotes/wiki/Plane)
ARCore中所有的内容，都要依托于平面类进行渲染。如Demo中的Android机器人，只有在检测到网格的地方才能放置。

ARCore中平面可分为水平朝上、朝下和非水平平面类型。Plane描述了对一个真实世界二维平面的认知，如平面的中心点、平面的x和z轴方向长度，组成平面多边形的顶点。检测到的平面还分为三种状态，分别是正在跟踪，可恢复跟踪和永不恢复跟踪。如果是没有正在跟踪的平面，包含的平面信息可能不准确。

两个或者多个平面还会被被自动合并成一个父平面。如果这种情况发生，可以通过子平面找到它的父平面

## [Anchor](https://github.com/huben2koishi/ARCore-LearningNotes/wiki/Anchor)
每个Android机器人的位置，就是一个Anchor。Anchor位置会随着手机或者摄像头的位置更新而更新，这样才能保持“位置不变”。

锚点描述了在现实世界中一个固定的位置和方向。为了保持在物理空间的固定位置，这个位置的数值描述将会随着ARCore对空间的理解的改进而更新。使用getPose()获取这个Anchor的当前数值位置，这个位置每次update()被调用的时候都可能改变，但不会自发地改变

## [Pose](https://github.com/huben2koishi/ARCore-LearningNotes/wiki/Pose)
Pose表示从一个坐标空间到另一个坐标空间位置不变的转换。Pose总是描述从对象本地坐标空间到世界坐标空间的转换

随着ARCore对环境的了解不断变化，它将调整坐标系模式以便与真实世界保持一致

这时，Camera和Anchor的位置（坐标）可能会发生明显的变化，以便它们所代表的物体处理恰当的位置

这意味着，每一帧图像都应被认为是在一个完全独立的世界坐标空间中。Anchor和Camera的坐标不应该在渲染帧之外的地方使用，如果需考虑到某个位置超出单个渲染框架的范围，则应该创建一个锚点或者应该使用相对于附近现有锚点的位置

## [ImageMetadata](https://github.com/huben2koishi/ARCore-LearningNotes/wiki/ImageMetadata)
提供了对Camera图像捕捉结果的元数据的访问

## [LightEstimate](https://github.com/huben2koishi/ARCore-LearningNotes/wiki/LightEstimate)
LightEstimate给我们提供了一个接口来查询当前帧的光环境。我们可以获取当前相机视图的像素强度，一个范围在（0.0，1.0）的值，0代表黑色，1代表白色。使用该光线属性绘制内容，显得更逼真。通过getLightEstimate()得到