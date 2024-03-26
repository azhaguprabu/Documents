### Docker using proxy
```

vim /etc/docker/daemon.json
{
 "proxies": {
   "default": {
     "httpProxy": "http://proxy.example.com:3128",
     "httpsProxy": "https://proxy.example.com:3129",
     "noProxy": "*.test.example.com,.example.org,127.0.0.0/8"
   }
 }
}

```

### docker using proxy while build and run exec
```
docker build --build-arg HTTP_PROXY="http://proxy.example.com:3128" .
docker run --env HTTP_PROXY="http://proxy.example.com:3128" redis
```

### Dockerfile proxy enable
```
vim Dockerfile

From Ubuntu:latest
ENV HTTP_PROXY="http://192.168.1.10:3621"
ENV HTTPS_PROXY="http://192.168.1.10:3621"

```