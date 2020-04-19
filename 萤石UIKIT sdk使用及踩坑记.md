#### 需求
需求是完成一个视频列表，里面是每个现场的实时视频，利用萤石sdk单窗口监控模式实现

#### 官方文档
[https://open.ys7.com/doc/zh/uikit/uikit\_javascript.html](https://open.ys7.com/doc/zh/uikit/uikit_javascript.html)

#### 使用方法
这里主要说「监控模式」
##### 这里可以使用CDN的方式，在index.html中全局引入
##### dom结构
需求是页面中有许多视频，于是根据接口返回的视频列表，动态渲染了视频容器
![dom](https://user-images.githubusercontent.com/38416128/79679979-39a5bb00-823d-11ea-94ab-f06871ffeed6.png)
    监控模式没有找到封面图的方法，所以手动写一个，根据视频播放状态判断是否显示
##### 视频播放方法
````
playVideo(vi, index) {
      this.curVideoIndex = index
      let id = `${vi.deviceSerial}-${index}`
      this.palyIdArr.push(id)
      if(this.palyIdArr.length > 1) {
        const dom = document.getElementById(this.palyIdArr[this.palyIdArr.length - 2])
        const ch = dom.childNodes
        dom.removeChild(ch[0])
      }
      
      this.player && this.pauseVideo()
      let domain = 'open.ys7.com'
      let definition = ''
      
      this.videoOnPlay = true
      
      this.player = new window.EZUIPlayer({
        id,
        url: `ezopen://${domain}/${vi.deviceSerial}/${vi.channelNo}${definition}.live`,
        autoplay: false,
        accessToken: vi.token,
        decoderPath: 'ezuikit',
        width: 200,
        height: 133,
        controls: true
      });

      this.player.play()
    }
````

#### 踩坑
✨报audio相关的错，可能是没有命中源码video的逻辑，可以打断点看一下参数是否传对了

✨多个视频切换播放：因为一个页面里要渲染多个canvas，在一个视频暂停后，播放另一个视频会出现当前播放的视频没有画面的情况。如果你的视频有声音，会发现视频只有声音没有画面。我开始以为是视频的问题，但发现单独播放是没有问题的。后来发现视频确实是成功播放了，但是是在上一个播放的视频容器里。

     我目前的解决办法：

     1.  检查你的视频id和容器id是否匹配
    
     2.  如果id没问题，那么需要操作dom，把上一个视频容器里渲染出的所有dom删除掉
    

✨视频正常播放，控制台却有报错 TypeError: Cannot read property ‘audioId’ of undefined
![error](https://user-images.githubusercontent.com/38416128/79680000-5b9f3d80-823d-11ea-9870-884fac95f1d8.png)
这是skd本身的容错bug
    调用视频的paly方法：play({}, {})
✨全屏方法：
    （这是文档的bug，文档写的极其不详细，想用啥方法可以去源码搜一下）

这里的全屏方法是**fullScreen()**，另外双击也可以命中全屏及退出全屏方法
