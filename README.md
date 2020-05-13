# lite-app-template

> 小程序模板

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report
```

For detailed explanation on how things work, checkout the [guide](http://vuejs-templates.github.io/webpack/) and [docs for vue-loader](http://vuejs.github.io/vue-loader).


使用最新版的mpvue开发的
主要针对微信小程序webview
脱坑之路
主要代码如下：
微信web-view（小程序内置h5）
h5端：引入微信jssdk/使用npm安装jssdk
<script type="text/javascript" src="https://res.wx.qq.com/open/js/jweixin-1.3.2.js"></script>
npm install weixin-js-sdk --save
import wx from 'weixin-js-sdk';
Vue.prototype.$wx = wx;
h5向小程序传值：
mounted（）{
	wx.miniProgram.getEnv(res => {
                	this.inLiteApp = res.miniprogram
    	})
}
方法: {
	if (this.inLiteApp == true) {
              		wx.miniProgram.navigateTo({
               			 url: '/pages/payment/index/main?orderId=' + this.detail.orderId + '&orderCode=' + this.detail.orderCode + '&amount=' + this.detail.stats[2].value
              		})
           	 }
}
小程序接收h5传过来的值(h5传过来的值直接从optios里取)
  async onLoad(options) {
    	this.payInfo = options
  }
小程序向h5传值：
<web-view :src="webviewUrl" @message="postMessage"/>
  onLoad(options) {
	// params为其他页面或者接口里接收到的数据
	const params = {
            		userId: this.userId, 
            		ticket: this.ticket,
            		appTicket: this.appTicket
          	}
    	this.webviewUrl = `https://mini.yangche51.com/#/home?parmas=${this.params}`;
  }
h5接收小程序传过来的值
  created() {
     wx.miniProgram.getEnv(res => {
        if (res.miniprogram) { //判断微信环境
          const queryUrl = this.getRequest()
          let urlParams = Object.assign(queryUrl, this.$route.query)  //参数合并  Object.assign（）是浅拷贝
          let wxParmas = JSON.parse(queryUrl.parmas)
          console.log('我接收到的参数', wxParmas)
          document.cookie = "app_ticket=" + wxParmas.appTicket;  //接收到的微信参数放入cookie里
          this.$store.dispatch("auth/saveToken", {  调取store里的user.js的auth/saveToken方法，更新数据
              token: wxParmas.ticket,
              remember: 365
          })
        }
    })
  },
methods: {
    getRequest() {
         var url = decodeURIComponent(location.search); // 对参数解码获取到'#'后面的参数
         var theRequest = new Object();
          if (url.indexOf("?") != -1) {  // 查找是否包含'?'
            var str = url.substr(1); // 截取字符串
            var strs = str.split("&"); // 分割字符串
            for(var i = 0; i < strs.length; i ++) {
                theRequest[strs[i].split("=")[0]] = unescape(strs[i].split("=")[1]); // unescape（）函数解码字符串
            }
           }
          return theRequest; //返回结果
    },
}
