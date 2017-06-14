# Cross-Domain
a lit solution for Cross Domain

同源策略：同协议、同端口、同域名

## 1.变更源
　　　　在a.name.com/a.html中
` document.domain = 'a.com';
 
var ifr = document.createElement('iframe');
ifr.src = 'http://b.name.com/b.html';
ifr.display = none;
document.body.appendChild(ifr);
 
ifr.onload = function(){
    var doc = ifr.contentDocument || ifr.contentWindow.document;
    //在这里操作doc，也就是b.html
    ifr.onload = null;
};
document.domain = 'name.com'; `

## 2.动态创建script
` function loadScript(url, func) {
  var head = document.head || document.getElementByTagName('head')[0];
  var script = document.createElement('script');
  script.src = url;
 
  script.onload = script.onreadystatechange = function(){
    if(!this.readyState || this.readyState=='loaded' || this.readyState=='complete'){
      func();
      script.onload = script.onreadystatechange = null;
    }
  };
 
  head.insertBefore(script, script[0]);
}
window.baidu = {
  sug: function(data){
    console.log(data);
  }
}
loadScript('https://www.baidu.com',function(){console.log('loaded')}); `

## 3.window.name + iframe window对象有个name属性，该属性有个特征：即在一个窗口(window)的生命周期内,窗口载入的所有的页面都是共享一个window.name的，每个页面对window.name都有读写的权限，window.name是持久存在一个窗口载入过的所有页面中的，并不会因新页面的载入而进行重置
a.com/a.html
` <!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title></title>
        <script>
            function getData(){
                //此时window.name已被修改为b.com/b.html页面设置的数据
                var iframe = document.getElementById('proxy');
                iframe.onload = function(){
                    var data = iframe.contentWindow.name;//获取iframe中window.name,也就是b.com/b.html页面设置的数据
                    alert(data);
                }
                iframe.src = 'about:block'; //赊着src的目的是为了让iframe与当前页面同源。src被修改后会重新load然后触发上面的onload
            }
        </script>
    </head>
    <body>
        <iframe id="proxy" src="b.com/b.html" onload="getData()"></iframe>
    </body>
</html> `

## 4.4.postMessage（HTML5中的XMLHttpRequest Level 2中的API）
a.com/index.html
` <!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title></title>
        <script>
            var iframe = document.getElementById('iframe');
            iframe.contentWindow.postMessage('我是a.com/index.hmtl的消息', '*');
        </script>
    </head>
    <body>
        <iframe id="iframe" src="b.com/index.html"></iframe>
    </body>
</html> `
b.com/index.html
` <script>
    window.onmessage = function(e){
        e = e || event;
        alert(e.data)
    }
</script> `

## 5.CORS（Cross-Origin Resource Sharing）
front-end
` function getHello() {
    var xhr = new XMLHttpRequest();
    xhr.open("post", "https://b.example.com/Test.ashx", true);
    xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");　　　
     
    xhr.onreadystatechange = function () {
        if (xhr.readyState == 4 && xhr.status == 200) {
            var responseText = xhr.responseText;
            console.info(responseText);
        }
    }
    xhr.send();
} `
back-end
` header('Access-Control-Allow-Origin:*')
`
## 6.JSONP
` function handleResponse(response){
    console.log('The responsed data is: '+response.data);
}
var script = document.createElement('script');
script.src = 'http://www.baidu.com/json/?callback=handleResponse';
document.body.insertBefore(script, document.body.firstChild); `

## 7.Nginx反向代理
前端调用的服务 /apis/xxxx/xxxx  和当前页是同源的，nginx来做一个代理到想要的地方，来实现跨域nginx.conf 配置一个反向代理路径
` location /apis {
    rewrite ^.+apis/?(.*)$ /$1 break;
    include uwsgi_params;
    proxy_pass http://www.baicu.com/xxxx
} `
