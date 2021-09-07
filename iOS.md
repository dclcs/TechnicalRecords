## UISlider 



UISlider 是用来控制一个范围内的连续值。

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gu29r1zcevj60ju06iaa402.jpg)

将一个Slider加到页面中

- 需要确认slider所表达的范围
- 可以选择配置slider的外观
- 链接一个或者多个action方法到slider上

### 与用户的交互

Slider 使用 Target-Action的设计模式来通知你的App什么时候移动slider。你可以通过注册`value Changed`事件来通知slider的值发生了改变。默认情况下，slider的值的改变是连续的，但是你可以将`isContinuous`设置为`false`来当用户释放slider的控制时到最后的值。

## AVAudioPlayer

与播放器事件相关的事件

#### AVAudioPlayerDelegate

- `func audioPlayerDidFinishPlaying(AVAudioPlayer, successfully: Bool)`
  - player : 播放器
  - successfully ： 播放器是否完整播放
- 



