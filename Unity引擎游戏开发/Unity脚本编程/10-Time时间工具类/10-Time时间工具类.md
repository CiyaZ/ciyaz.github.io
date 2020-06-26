# Time时间工具类

Unity中提供了一个`Time`类，封装了一些和时间相关的数据接口。由于有些实在是太常用了，所以我们单独介绍一下。

注：以下属性单位均为秒（s）。

## Time.time

游戏开始到现在的总时间，会随着游戏暂停而暂停。

## Time.timeSinceLevelLoad

当前场景加载到现在的总时间，会随着游戏暂停而暂停。

## Time.deltaTime

上一帧到现在的时间。

## Time.fixedDeltaTime

物理时钟下，到上一次时钟触发的间隔时间。

## Time.timeScale

时间缩放，默认值为`1`，可以用来加速或减速游戏的运行，做出类似子弹时间的特效。
