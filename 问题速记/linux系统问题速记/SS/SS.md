# SS

install
```
pip3 install shadowsocks
```

demo config file:
```json
{
  "server":"74.82.221.44",
  "server_port":7078,
  "local_address":"127.0.0.1",
  "local_port":1080,
  "password":"root",
  "timeout":300,
  "method":"aes-256-cfb",
  "fast_open":true
}
```

run ss:
```
ssserver -c [ur_conf_file_path] -d [start|stop]
```
