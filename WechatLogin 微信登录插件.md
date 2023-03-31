# WechatLogin 微信登录插件

------

### 简介：

对于微信登录的开发总是需要引入其他的SDK进行二次开发，该插件则可以实现快捷方便性，可以一次启动，一直使用。



### 使用：

##### Linux

```
nohup ./wechat_login_linux -c config.yaml > start.log 2>&1 &
```

##### Window

```
.\wechat_login_windows config.yaml
```



### 配置:

```yaml
#基础配置
app:
  name: "wechat_login"
  mode: "dev"
  port: 8080
  version: "0.0.1"
  start_time: "2023-03-30"
  machine_id: 1

#日志记录
log:
  level: "debug"
  filename: "wechat_login.log"
  max_size: 200
  max_age: 30
  max_backups: 7

#Mysql配置
mysql:
  host: "127.0.0.1"
  port: 3306
  user: "root"
  password: "xxxx"
  dbname: "wechat"
  max_open: 200
  max_idle: 10

#Redis配置
redis:
  host: "127.0.0.1"
  password: ""
  pool: 20
  port: 6379
  db: 0

#微信公众号相关配置
wechat:
  appID: "xxxx" //
  appSecret: "xxxx"
  token: "xxxx"
```



### Demo网页：

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>wechat_login</title>
    <style>
        /* 让内容居中显示 */
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }

        /* 设置输入框和发送按钮的样式 */
        .input-container {
            display: flex;
            align-items: center;
            justify-content: center;
            margin-bottom: 20px;
        }

        input[type=text] {
            padding: 10px;
            font-size: 16px;
            border-radius: 5px;
            border: none;
            box-shadow: 1px 1px 3px rgba(0, 0, 0, 0.1);
            width: 300px;
            margin-right: 10px;
        }

        button {
            padding: 10px 20px;
            font-size: 16px;
            border-radius: 5px;
            border: none;
            background-color: #4CAF50;
            color: white;
            cursor: pointer;
        }

        button:hover {
            background-color: burlywood;
        }

        .green {
			background-color: #2EB9C3;
		}
		.yellow {
			background-color: #F5A623;
			color: black;
		}
		.red {
			background-color: #D0021B;
		}

        /* 设置信息接收框的样式 */
        textarea {
            padding: 10px;
            font-size: 16px;
            border-radius: 5px;
            border: none;
            box-shadow: 1px 1px 3px rgba(0, 0, 0, 0.1);
            width: 400px;
            height: 300px;
        }

        textarea[readonly] {
            background-color: #EEE;
            cursor: not-allowed;
        }
    </style>
</head>

<body>
    <div class="input-container">
        <button class="yellow" onclick="check()">检查状态</button>
        <button class="green" onclick="send()">验证登录</button>
        <button class="red" onclick="del()">注销登录</button>
    </div>
    <textarea id="info-box" name="info-box" readonly></textarea>
    <div id="image"></div>
    <script>
        const infoBox = document.getElementById("info-box");
        var token;

        var ws = new WebSocket("ws://wechat.tcmree.cn/socket");
        // var ws = new WebSocket("ws://localhost/socket");

        //连接打开时触发 
        ws.onopen = function (evt) {
            infoBox.value += "Connection open ...\n";
        };
        //接收到消息时触发  
        ws.onmessage = function (evt) {
            console.log(evt);
            var message = JSON.parse(evt.data);
            if (message.type == "image") {
                // 获取要追加图片的 div 元素
                var imgDiv = document.getElementById("image");

                // 创建一个新的 img 元素，并设置其 src 属性为 Base64 编码的图片数据
                var imgEle = document.createElement("img");
                imgEle.src = "data:image/png;base64," + message.data;

                // 将新的 img 元素追加到 div 元素中
                imgDiv.appendChild(imgEle);
            }
            else if (message.type == "token") {
                infoBox.value += "Received Token: " + message.data + "\n";
                token = message.data;
                // 存储token
                localStorage.setItem('token', token);
            }
            else if (message.type == "verify") {
                if (message.data == "200") {
                    infoBox.value += "登录成功！\n";

                }
                else {
                    infoBox.value += "登录失败！请扫码关注登录...\n";
                }
            }
            else if (message.type == "status") {
                if (message.data == "200") {
                    infoBox.value += "当前状态：登录成功\n";

                }
                else {
                    infoBox.value += "当前状态：未登录\n";
                }
            }
        };
        //连接关闭时触发  
        ws.onclose = function (evt) {
            infoBox.value += "Connection closed...\n";
        };

        function send() {
            ws.send("verify");
        }

        function check() {
            token = localStorage.getItem('token');
            if (token != null) {
                wx.send(token)
            }
            else {
                infoBox.value += "当前状态：未登录\n";
            }
            
        }

        function del() {
            localStorage.removeItem('token');
            infoBox.value += "已注销登录信息\n";
            check();
        }
    </script>
</body>

</html>
```

