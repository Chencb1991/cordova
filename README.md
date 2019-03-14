# cordova笔记
* 安装
```
npm install -g cordova 
```
* 创建项目
```
#cordova create path（文件夹名字）id（应用id名字）name（应用名字）
exmple：cordova create myproject com.example.myproject testpro
```

* 判断平台
```
cordova plugin add cordova-plugin-device
if(device.platform==='iOS'){
    StatusBar.backgroundColorByHexString("transparent");
    StatusBar.overlaysWebView(false);   //不被webView覆盖
}

```
> https://blog.csdn.net/u011127019/article/details/56281444

* cd 目录
* 安装平台
```
#cordova platform add ios
#cordova platform add android 
```

* 检查你当前平台设置状况
```
cordova platform ls （检查你当前平台设置状况）
```
* 打包
```
cordova build android
```



## 结合vue
* 直接在项目文件中安装vue_cli与wwww文件平级
* 配置输出打包目录为www
```
 outputDir: "../www",
 assetsDir: "./",
```
* index.html引入cordova.js和自定义index.js并加入csp策略
```
<meta http-equiv="Content-Security-Policy" content="default-src *  data: gap: https://ssl.gstatic.com 'unsafe-eval'; style-src * 'unsafe-inline'; media-src *">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="format-detection" content="telephone=no">
    <meta name="msapplication-tap-highlight" content="no">
 <script type="text/javascript" src="cordova.js"></script>
 <script type="text/javascript" src="index.js"></script>
 ```
 * 安装cordova插件(部分)
 ```
 网络连接：cordova plugin add cordova-plugin-network-information
相机插件：cordova plugin add cordova-plugin-camera
白名单插件：cordova plugin add cordova-plugin-whitelist
通讯录插件：cordova plugin add cordova-plugin-contacts
Cordova InAppBrowser： cordova plugin add cordova-plugin-inappbrowser
地理位置：CordovaProject>cordova plugin add cordova-plugin-geolocation
cordova plugin add cordova-plugin-x-toast
```
* 方法使用index.js
```
document.addEventListener("deviceready", onDeviceReady, false);
function onDeviceReady() {
    navigator.camera.cleanup(onSuccess, onFail,{
        quality: 50,//图片质量
          allowEdit: true,//允许编辑
          correctOrientation:true,
          cameraDirection:1,//前置拍照
          //targetWidth:468,
          //targetHeight:300,
          destinationType: Camera.DestinationType.DATA_URL
    });
        function onSuccess() {
            console.log("Camera cleanup success.")
        }
        function onFail(message) {
            console.log('Failed because: ' + message);
        }
        
   
    document.addEventListener("backbutton", onBackKeyDown, false);
        function onBackKeyDown() {
            window.plugins.toast.showLongCenter('再点击一次退出!');  
            document.removeEventListener("backbutton", onBackKeyDown, false); //注销返回键  
            document.addEventListener("backbutton", exitApp, false);//绑定退出事件
            //3秒后重新注册  
            var intervalID = window.setInterval(  
            function() {  
              window.clearInterval(intervalID);  
              document.removeEventListener("backbutton", exitApp, false); // 注销返回键  
              document.addEventListener("backbutton", onBackKeyDown, false); //返回键  
            },  
            3000
              );  
            function exitApp(){
            navigator.app.exitApp();
            }
        }

        var app3 =[];
        var contactFileds = ["phoneNumbers"];
        //filter制定为空或不指定返回所有联系人列表
        var options = { filter: "", multiple: true };
        navigator.contacts.find(contactFileds, onSuccess, onError, options);
        function onSuccess(contacts) {     
        //创建联系人对象数组
              var contactsArr = [];
              for (var i = 0; i < contacts.length; i++) {
                //创建联系人对象        
                var currContact = {};
                //设置联系人名称
                currContact.displayName = contacts[i].displayName;
                //设置联系人电话号码
                var phoneNumbers = [];
                if(contacts[i].phoneNumbers != null){
                    for(var j=0;j<contacts[i].phoneNumbers.length;j++){
                      phoneNumbers.push(contacts[i].phoneNumbers[j].value);
                    }
                 }
                 currContact.phoneNumbers = phoneNumbers;
                 contactsArr.push(currContact);
              }

            sessionStorage.setItem('teljoin',JSON.stringify(contactsArr));
           
        }
        function onError(err) {
            app3 = JSON.stringify(err);
        }
     //网络链接
        document.addEventListener("offline", onOffline, false);
        document.addEventListener("online", onOnline, false);
     
        function onOffline() {
           //window.plugins.toast.showLongCenter('网络链接失败');
           alert('网络链接失败');
        }

        function onOnline() {
            console.log('网络链接正常');
        }
     //获取经纬度
     navigator.geolocation.getCurrentPosition(iponSuccess, iponError);
     function iponSuccess(position){
         //alert('纬度:'+ position.coords.latitude+'\n'+'经度:'+ position.coords.longitude+'\n');
         sessionStorage.setItem('dwdlatitude',position.coords.latitude);
         sessionStorage.setItem('longitude',position.coords.longitude);
     }      
     //定位数据获取失败响应
     function iponError(error) {
        console.log(error.message);
    } 
    navigator.splashscreen.hide();//隐藏启动页面


}
```

* 生成签名：
```
keytool -genkey -v -keystore dwd.keystore -alias dwd -keyalg RSA -keysize 2048 -validity 20000
```
> keystore   后面是数字签名证书，  --alias 后面是别名    storePassword   后面是密钥库口令    password 后面是密钥口令 

* 签名打包：
```
cordova build android --release -- --keystore="dwd.keystore" --alias=dwd --storePassword=? --password=?
```


* 内页返回键返回上一页，tab退出问题解决

```
methods:{
 onBackKeyDown() {
      var that = this;
            window.plugins.toast.showLongCenter('再点击一次退出!');  
            document.removeEventListener("backbutton", this.onBackKeyDown, false); //注销返回键  
            document.addEventListener("backbutton", this.exitApp, false);//绑定退出事件
            //3秒后重新注册  
            var intervalID = window.setInterval(  
              this.functions(),
            3000
              );  
            this.exitApp()
        },
    functions() {  
                window.clearInterval(intervalID);  
                document.removeEventListener("backbutton", this.exitApp, false); // 注销返回键  
                document.addEventListener("backbutton", this.onBackKeyDown, false); //返回键  
              }, 
    exitApp(){
            navigator.app.exitApp();
        }
},
 watch: {
      $route: {
        handler: function(val, oldVal){
          if(val.name=='mains'||val.name=='loan'||val.name=='user'){
          document.addEventListener("backbutton", this.onBackKeyDown, false);
          }else{
            //移除监听
          document.removeEventListener('backbutton', this.onBackKeyDown, false);
          document.removeEventListener("backbutton", this.exitApp, false); // 注销返回键 
          // document.removeEventListener('backbutton', this.pageBack, false)
          }
        },
        // 深度观察监听
        deep: true
      }
    }
   ```











 
 ----------------------------------------------------------------------------
 
 ## cordova ios
 
 > 在完成苹果开发者和已申请签名证书和描述文件的情况下以及有一台苹果电脑，支持xcode
 
 * 安装cordova (同上)
 * 添加ios平台
 * corrdova build ios(可略)
 * 找到平台下 /platform/ios/xx.xcodeproj 文件直接双击打开xcode
 * 编写功能插件，安装插件等
 ```
 document.addEventListener("deviceready", onDeviceReady, false);
function onDeviceReady() {
    var app3 =[];
    var contactFileds = [""];
    //filter制定为空或不指定返回所有联系人列表
    var options = { filter: "", multiple: true };
    navigator.contacts.find(contactFileds, onSuccess, onError, options);
    function onSuccess(contacts) {
            //创建联系人对象数组
              var contactsArr = [];
              for (var i = 0; i < contacts.length; i++) {
                //创建联系人对象        
                var currContact = {};
                //设置联系人名称
                currContact.displayName = contacts[i].displayName;
                //设置联系人电话号码
                var phoneNumbers = [];
                if(contacts[i].phoneNumbers != null){
                    for(var j=0;j<contacts[i].phoneNumbers.length;j++){
                      phoneNumbers.push(contacts[i].phoneNumbers[j].value);
                    }
                 }
                 currContact.phoneNumbers = phoneNumbers;
                 contactsArr.push(currContact);
              }

            localStorage.setItem('teljoin',JSON.stringify(contactsArr));
            
        }
    function onError(err) {
            app3 = JSON.stringify(err);
        }


     //获取经纬度
     navigator.geolocation.getCurrentPosition(iponSuccess, iponError);
     function iponSuccess(position){
         console.log('纬度:'+ position.coords.latitude+'\n'+'经度:'+ position.coords.longitude+'\n');
         localStorage.setItem('dwdlatitude',position.coords.latitude);
         localStorage.setItem('longitude',position.coords.longitude);
     }      

     //定位数据获取失败响应
     function iponError(error) {
        console.log(error.message);
    } 
  
}
```

 * 添加权限(部分)  /platform/ios/项目名/项目名-info.plist里
 
 ```
    <key>NSLocationWhenInUseUsageDescription</key>
    <string>需要访问您的GPS权限</string>
    <key>NSLocationAlwaysUsageDescription</key>
    <string>需要访问您的GPS权限</string>
    <key>NSCameraUsageDescription</key>
    <string>需要摄像头权限</string>
    <key>NSContactsUsageDescription</key>
    <string>需要通讯录权限</string>
    <key>NSLocationUsageDescription</key>
    <string>需要您的位置权限</string>
    
    
    <!-- 相册 -->   
<key>NSPhotoLibraryUsageDescription</key>   
<string>App需要您的同意,才能访问相册</string>   
<!-- 相机 -->   
<key>NSCameraUsageDescription</key>   
<string>App需要您的同意,才能访问相机</string>   
<!-- 麦克风 -->   
<key>NSMicrophoneUsageDescription</key>   
<string>App需要您的同意,才能访问麦克风</string>   
<!-- 位置 -->   
<key>NSLocationUsageDescription</key>   
<string>App需要您的同意,才能访问位置</string>   
<!-- 在使用期间访问位置 -->   
<key>NSLocationWhenInUseUsageDescription</key>   
<string>App需要您的同意,才能在使用期间访问位置</string>   
<!-- 始终访问位置 -->   
<key>NSLocationAlwaysUsageDescription</key>   
<string>App需要您的同意,才能始终访问位置</string>   
<!-- 日历 -->   
<key>NSCalendarsUsageDescription</key>   
<string>App需要您的同意,才能访问日历</string>   
<!-- 提醒事项 -->   
<key>NSRemindersUsageDescription</key>   
<string>App需要您的同意,才能访问提醒事项</string>   
<!-- 运动与健身 -->   
<key>NSMotionUsageDescription</key> <string>App需要您的同意,才能访问运动与健身</string>   
<!-- 健康更新 -->   
<key>NSHealthUpdateUsageDescription</key>   
<string>App需要您的同意,才能访问健康更新 </string>   
<!-- 健康分享 -->   
<key>NSHealthShareUsageDescription</key>   
<string>App需要您的同意,才能访问健康分享</string>   
<!-- 蓝牙 -->   
<key>NSBluetoothPeripheralUsageDescription</key>   
<string>App需要您的同意,才能访问蓝牙</string>   
<!-- 媒体资料库 -->   
<key>NSAppleMusicUsageDescription</key> 
 <string>App需要您的同意,才能访问媒体资料库</string>  

 ```
 
  > 或者直接在xcode里面设置  ，如果不填写设置，无权限，相机闪退黑屏等情况，插件失效等
 
  * 必须有开发者帐号才能打包，免费的可能导不出安装文件，可以链接手机调试功能。
 
  * 情况1：有时候出现不如保存数据到本地，需要添加<preference name="BackupWebStorage" value="local"/>,最好使用localStorage
  * 情况2：ios里面不支持cordova里的contacts里的displayName属性
  
  ```
  currContact.displayName = contacts[i].displayName;//安卓
  currContact.displayName = contacts[i].name.formatted;//ios
                ```
                 
  

