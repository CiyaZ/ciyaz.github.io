# v2ray-内网穿透

防火墙技术和内网穿透技术，是信息对抗中一个重要方面。内网穿透从最初的简单加密流量，到设计容易让防火墙混淆的协议，以及代理服务器多节点的部署，都随着技术和计算力发展被防火墙抵挡了下来。

然而v2ray能够把流量伪装成websocket数据来骗过防火墙，而websocket数据甚至能通过CDN转发，这一设计其实非常巧妙，因为它基本抹消了穿透流量的特征，混入普通数据的海洋中，目前来看是没有什么好的应对方式了。

项目地址：[https://github.com/v2ray](https://github.com/v2ray)

## 配置v2ray服务端例子

下面是一个配置v2ray服务端的例子，可供实验时参考。

```json
{
    "inbounds": [
        {
            "port": <端口值>,
            "listen": <IP字符串>,
            "protocol": "vmess",
            "settings": {
                "clients": [
                    {
                        "id": <随机生成的id值>,
                        "level": 1,
                        "alterId": 64
                    }
                ]
            },
            "streamSettings": {
                "network": "ws",
                "wsSettings": {
                    "path": "/"
                }
            }
        }
    ],
    "outbounds": [
        {
            "protocol": "freedom",
            "settings": {}
        },
        {
            "protocol": "blackhole",
            "settings": {},
            "tag": "blocked"
        }
    ],
    "routing": {
        "rules": [
            {
                "type": "field",
                "ip": [
                    "geoip:private"
                ],
                "outboundTag": "blocked"
            }
        ]
    }
}
```
