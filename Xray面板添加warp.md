## 一、安装Cloudflare 的 WARP

根据[官网教程](https://pkg.cloudflareclient.com/)安装适合自己系统的warp，下面以Ubuntu系统为例：  
1、添加 cloudflare gpg 密钥  
	`curl -fsSL https://pkg.cloudflareclient.com/pubkey.gpg | sudo gpg --yes --dearmor --output /usr/share/keyrings/cloudflare-warp-archive-keyring.gpg`  
2、将此存储库添加到您的 apt 存储库中  
	`echo "deb [signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflare-client.list`  
3、安装  
	`sudo apt-get update && sudo apt-get install cloudflare-warp`

## 二、首次使用WARP

[官网教程](https://developers.cloudflare.com/warp-client/get-started/linux/)  
1、注册客户端  
	`warp-cli registration new`  
2、设置为代理模式，默认端口40000，客户端将通过 SOCKS5 代理来使用 WARP 流量（一定要先设置，不然虚拟机会失联）  
	`warp-cli mode proxy`  
3、连接WARP  
	`warp-cli connect`  
4、确认代理是否成功，成功的话输出会包含warp=on  
	`curl --socks5 127.0.0.1:40000 https://www.cloudflare.com/cdn-cgi/trace`  
下面是检查代理状态命令（可选）：  
	`warp-cli status`

## 三、修改配置面板

以[3x-ui面板](https://github.com/MHSanaei/3x-ui/blob/main/README.zh_CN.md)为例，进入面板后点击Xray设置➡高级配置，复制另存为json文件做备份，建议在本地用支持语法高亮的编辑器修改避免改错，注意是否要添加逗号。修改的思想是在出站规则添加socks5-warp，在路由规则添加需要warp代理的域名或geosite，并把出口指向socks5-warp，修改完成后保存配置➡重启面板。下面是完整配置：

```json
{
  "log": {
    "access": "none",
    "dnsLog": false,
    "error": "",
    "loglevel": "warning",
    "maskAddress": ""
  },
  "api": {
    "tag": "api",
    "services": [
      "HandlerService",
      "LoggerService",
      "StatsService"
    ]
  },
  "inbounds": [
    {
      "tag": "api",
      "listen": "127.0.0.1",
      "port": 62789,
      "protocol": "dokodemo-door",
      "settings": {
        "address": "127.0.0.1"
      }
    }
  ],
  "outbounds": [
    {
      "tag": "direct",
      "protocol": "freedom",
      "settings": {
        "domainStrategy": "AsIs",
        "redirect": "",
        "noises": []
      }
    },
    {
      "tag": "blocked",
      "protocol": "blackhole",
      "settings": {}
    },
    {
      "tag": "socks5-warp",
      "protocol": "socks",
      "settings": {
        "servers": [
          {
            "address": "127.0.0.1",
            "port": 40000
          }
        ]
      }
    }
  ],
  "policy": {
    "levels": {
      "0": {
        "statsUserDownlink": true,
        "statsUserUplink": true
      }
    },
    "system": {
      "statsInboundDownlink": true,
      "statsInboundUplink": true,
      "statsOutboundDownlink": true,
      "statsOutboundUplink": true
    }
  },
  "routing": {
    "domainStrategy": "AsIs",
    "rules": [
      {
        "type": "field",
        "inboundTag": [
          "api"
        ],
        "outboundTag": "api"
      },
      {
        "type": "field",
        "outboundTag": "blocked",
        "ip": [
          "geoip:private",
          "geoip:cn"
        ]
      },
      {
        "type": "field",
        "outboundTag": "blocked",
        "protocol": [
          "bittorrent"
        ]
      },
      {
        "type": "field",
        "outboundTag": "socks5-warp",
        "domain": [
          "geosite:reddit",
          "ipinfo.io"
        ]
      }
    ]
  },
  "stats": {}
}
```
&nbsp;
*参考链接（有 ARM 机器的安装方法）：*[使用 WARP 解锁 ChatGPT 访问](https://blog.liqiye.com/posts/1831392938/index.html)