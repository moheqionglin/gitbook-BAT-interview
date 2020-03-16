## 基本语句
https://www.runoob.com/w3cnote/flex-grammar.html


- 语句 wx:if, wx:for 一般和<block>配合，因为不渲染block
	- for  [wx:for][1] 
		- ```<block wx:for="{{varName}}">i {{customName}}</block>``` 一般wx:for用<block>标签配合，因为不渲染block。 
			- ```<block wx:for="{{varName}}" wx:for-itme="customName" wx:for-index="i" ></block>``` 一般循环套循环的时候才会给外层的for起一个别名。
			- <block wx:for="{{}}" wx:key> 提高虚拟dom的diff算法
				
		- 支持类似于 for(int i =0; i < 10; i ++) ```<block wx:for="{{9}}"></block>```
		- 支持字符串遍历 ```<block wx:for="abcd"></block>```
		- 支持对象遍历 ```<block wx:for="{{array}}"></block>```
	- ```<block wx:if="{{var}}"></block>	```
	- ```<block wx:elif="{{var2}}"><block><block wx:else><block>```
	- 如果隐藏显示切换频率很高用 hidden属性，如果切换频率很低用wx:if
-	数据双向绑定，视图渲染
	1. 视图触发model的改变的时候调用this.setData({}) 视图会触发刷新动作。	

-	page.json 覆盖全局的 app.json

-	双线程和生命周期回调
	-	App onLaunch （只执行一次）-> onShow -> onHide -> onError 
	-	Page onLoad（页面生命周期内只执行一次） -> onShow -> onReady -> onHide -> onUnLoad

-	{{}} 里面不能调用js函数，只能调用wxs
	-	[wxs][2] 可以定义一个wxs文件
	-  一般定义一些工具，比如时间戳格式化，价格格式化。
	
-	内联
 	-	image text
- 权重
	- [element 1] < [.class 10] < [#id 100] < [style="" 1000] < [!important 无穷大]
	- 自定义组件内部的class样式 和 外部样式 互相不会产生干扰。 小程序 组件内部不允许使用 ID选择器，element选择器，属性选择器。
	- 默认是 Componetn({options: {styleIsolation: "isolate" }}) 所以相互不影响， shared 就会相互影响。
- wxss 尺寸新单位
	- rpx 自适应屏幕尺寸. 2rpx ≈ 1px， 所以 100px = 200rpx。 可以应用到容器大小，也可以用在字体大小上。 
		- 在iphone6 上 屏幕宽度 375排序， 共有750个物理像素。 1rpx = 0.5px
		- iphone5 1rpx = 0.42px
		- iphone6 plus 1rpx = 0.552px

	- 字体尺寸 rem
 
 - 事件 event

 	- target : 内部view， currentTarget：外层的view
 	- wxml里面的发生事件的时候，传递参数 <x data-xxx=""></x> 然后在 event.target.dataset
 	- 事件冒泡，事件捕获： 
 		- capture-bind:捕获事件顺序是 从【外层组件 ---> 内层组件】 一层层事件传递。 执行事件绑定行数的顺序是 反过来的。	 
 		- capture-catch: 阻止事件往内层传递。
 - 组件 和 页面的数据传递
	- 数据传递 properties: 
	- 样式传递 externalClasses: 
	- 标签 slot
	- 自定义事件传递

## 布局
### 水平垂直居中
#### 块级元素水平垂直居中

```
<view class="outter">
	<view>xxx</view>
</view>


.outter {
  display: flex;
  align-items: center; //交叉轴的剧种方式
  justify-content: center;//主轴居中方式
}

``` 
#### 文本水平垂直居中
```
<view class="outter">
	<text>xxx</text>
</view>
或者
<view class="outter">
	xxx
</view>

.outter{
	text-align:center;
	height: 120rpx;
	line-height: 120rpx;
	
}

```
### 形状
```
圆形
 width: 52rpx;
  border-radius: 50%;
  border: 2rpx solid #33cd5f
```
### box-sizing:  border-box
#### 文本自动换行
```
//单行
  width: 30%;
  text-overflow:ellipsis;
  word-wrap:break-word;
  white-space: nowrap;
  overflow: hidden;
  
  //多行
  overflow: hidden;
text-overflow: ellipsis;
display: -webkit-box;
-webkit-line-clamp: 2;/*几行，这里是2行*/
-webkit-box-orient: vertical;
 
```		
### 上传图片
```
<view class='uploader-img' wx:if="{{pics}}">
        <block wx:for="{{pics}}" wx:key="index">
          <view class='uploader-list' >
            <image src='{{item}}' data-index="{{index}}" mode="scaleToFill" bindtap='previewImg1' />
            <image class='delete' data-index="{{index}}" src='../../../images/common/delete.png' mode="widthFix" bindtap='deleteImg'/>
          </view>
        </block>
        
        <view class='upAdd' bindtap='chooseImg'>
          <image src='../../../images/common/upload-pic.png' mode="widthFix"/>
        </view>
      </view>
      
      
      
.uploader-img{
   display: flex;
   justify-content: flex-start;
   flex-wrap: wrap;
}
.uploader-img image{
  width: 160rpx;
  height: 160rpx;
} 
.uploader-list{
  position: relative;
  width: 160rpx;
  height: 160rpx;
  box-sizing: border-box;
  background-color: #ff0;
  margin: 40rpx 20rpx;
}
.uploader-list .delete{
  width: 40rpx;
  height: 40rpx;
  position: absolute;
  top: -10px;
  right: -10px;
  z-index: 100;
}



data: {
    pics: ['../../../images/22/191222150005929227.jpeg',
      '../../../images/22/191222150012374090.jpeg',
      '../../../images/22/191222150027135881.jpeg'],//图片
  },

previewImg: function (e) {

    var index = e.target.dataset.index;//当前图片地址
    var imgArr = e.target.dataset.list;//所有要预览的图片的地址集合 数组形式
    console.log(index, imgArr)
    wx.previewImage({
      current: imgArr[index],
      urls: imgArr,
    })
  },//上传图片开始
  // 预览图片
  previewImg1: function (e) {
    //获取当前图片的下标
    var index = e.currentTarget.dataset.index;
    //所有图片
    var pics = this.data.pics;
    wx.previewImage({
      //当前显示图片
      current: pics[index],
      //所有图片
      urls: pics
    })
  },
  chooseImg: function (e) {
    var that = this, pics = this.data.pics;
    console.log(pics);
    if (pics.length < 3) {
      wx.chooseImage({
        count: 3, // 最多可以选择的图片张数，默认9
        sizeType: ['original', 'compressed'], // original 原图，compressed 压缩图，默认二者都有
        sourceType: ['album', 'camera'], // album 从相册选图，camera 使用相机，默认二者都有
        success: function (res) {
          // 返回选定照片的本地文件路径列表，tempFilePath可以作为img标签的src属性显示图片
          var tempFilePaths = res.tempFilePaths;
          // wx.showToast({
          //   title: '正在上传...',
          //   icon: 'loading',
          //   mask: true,
          //   duration: 10000
          // });
          for (var i = 0; i < tempFilePaths.length; i++) {
            pics.push(tempFilePaths[i]);
          }
          console.log(pics);
          that.setData({
            pics: pics
          })
        },
      });
    } else {
      wx.showToast({
        title: '最多上传3张图片',
        icon: 'none',
        duration: 3000
      });

    }
  },
  // 删除图片
  deleteImg: function (e) {
    var that = this;
    var pics = this.data.pics;
    var index = e.currentTarget.dataset.index;
    pics.splice(index, 1);
    console.log(pics)
    this.setData({
      pics: pics,
    })
  },
```  

### 微信获取上一个页面数据，和返回上一页
```
let pages = getCurrentPages(); //获取当前页面js里面的pages里的所有信息。
      let prevPage = pages[pages.length - 2];
      var supplier = prevPage.data.suppliers.list.find(it => it.id == options.id)
      
      
      wx.navigateBack({
      delta: 1
    })
```	
### 下拉刷新
```
"enablePullDownRefresh": true

onPullDownRefresh: function () {
 
  },
```
### 坑1 scroll-view高度设置没用
```
小程序的scroll-view 高度必须动态
<scroll-view style="height: {{scrollViewHeight}}px" scroll-y="true">
  <!-- scroll-view里面的内容 -->
</scroll-view>
 
 计算

onLoad: function(option) {

        // 先取出页面高度 windowHeight
        wx.getSystemInfo({
            success: function(res) {
                that.setData({
                    windowHeight: res.windowHeight
                });
            }
        });

        // 然后取出navbar和header的高度
        // 根据文档，先创建一个SelectorQuery对象实例
        let query = wx.createSelectorQuery().in(this);
        // 然后逐个取出navbar和header的节点信息
        // 选择器的语法与jQuery语法相同
        query.select('#navbar').boundingClientRect();
        query.select('#header').boundingClientRect();

        // 执行上面所指定的请求，结果会按照顺序存放于一个数组中，在callback的第一个参数中返回
        query.exec((res) => {
            // 分别取出navbar和header的高度
            let navbarHeight = res[0].height;
            let headerHeight = res[1].height;

            // 然后就是做个减法
            let scrollViewHeight = this.data.windowHeight - navbarHeight - headerHeight;

            // 算出来之后存到data对象里面
            this.setData({
                scrollViewHeight: scrollViewHeight
            });
        });
    }
    
```

## 小程序 onShow， 即使是弹出框，关闭也会触发。
## 小程序 placeholder/输入内容不随textarea组件滚动
```
# 注意这里必须要用 wx:if + focus 要不然要点击两下才会拉起键盘输入
<view wx:if="{{item.hiddenTextarea}}" class="H4" style="height: 120rpx; margin-top: 20rpx" data-id="{{item.id}}" catchtap="clickCommentView">{{ productComments[index].comment|| '满意就好无保留的夸一夸吧~'}}</view>
<textarea fixed placeholder="满意就好无保留的夸一夸吧~" placeholder-class="H4" data-id = "{{item.productId}}" bindinput="typeComment" bindblur ="blurCommentTa" class="H3" wx:if="{{!item.hiddenTextarea}}" focus="{{!item.hiddenTextarea}}"></textarea>

```

[1]: https://developers.weixin.qq.com/miniprogram/dev/reference/wxml/list.html
[2]: https://developers.weixin.qq.com/miniprogram/dev/reference/wxs/