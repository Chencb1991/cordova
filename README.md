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

 
 


