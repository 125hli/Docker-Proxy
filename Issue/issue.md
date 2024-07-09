<div style="text-align: center"></div>
  <p align="center">
  <img src="https://github.com/dqzboy/Docker-Proxy/assets/42825450/c187d66f-152e-4172-8268-e54bd77d48bb" width="230px" height="200px">
      <br>
      <i>部署Docker Proxy服务和后期使用相关等问题总结.</i>
  </p>
</div>

---

[Docker Proxy-交流群](https://t.me/+ghs_XDp1vwxkMGU9) 

---

## 👨🏻‍💻 问题总结

#### 1、无法通过UI界面删除某一镜像的TAG
- [x] **已知问题：** 使用`registry`作为代理缓存时不支持删除

- [x] **issues：** [#3853](https://github.com/distribution/distribution/issues/3853)

#### 2、搭建好了，但是国内拉取速度不理想
- [x] **已知问题：** 你的国外服务器到国内的网络线路不理想

- [x] **解决方案：** 
  -  (1) 开启服务器BBR，优化网络性能(效果有限)
  - (2) 更换对国内网络线路优化更好的服务器

#### 3、registry 镜像缓存多少时间，如何调整
- [x] **已知问题：** 默认缓存`168h`，也就是`7天`。修改配置文件`proxy`配置部分`ttl`调整缓存时间

#### 4、使用镜像加速拉取`hub`公共空间下的镜像时如何不添加`library`

- 此方案来自交流群里大佬提供，通过nginx实现并实测
```shell
location ^~ / {
    if ($request_uri ~  ^/v2/([^/]+)/(manifests|blobs)/(.*)$) {
            # 重写路径并添加 library/
            rewrite ^/v2/(.*)$ /v2/library/$1 break;
    }

    proxy_pass http://127.0.0.1:51000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header REMOTE-HOST $remote_addr;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
    proxy_http_version 1.1;
    add_header X-Cache $upstream_cache_status;
}
```

#### 5、拉取镜像报错 `tls: failed to verify certificate: x509: certificate signed by unknown authority`
- [x] **已知问题：** 证书问题。表示证书是由一个未知的或不受信任的证书颁发机构（CA）签发的。

#### 6、通过docker-compose部署，如何设置Proxy认证
- [x] **已知问题：** 查看教程：[自建Docker镜像加速服务](https://www.dqzboy.com/8709.html)

#### 7、对于服务器的规格要求，内存、CPU、磁盘、带宽网络等
- [x] **已知问题：** 建议最低使用1C1G的服务器，磁盘大小取决于你拉取镜像的频率以及保存镜像缓存的时长决定(默认缓存7天,部署时可自定义)；如果对拉取速度有要求，最好选择针对中国大陆进行网络优化的服务器

#### 8、缓存在本地磁盘上的数据是否会自动清理？如何判断缓存是否被清理？
- [x] **已知问题：** Registry会定期删除旧内容以节省磁盘空间。下面为官方原文解释：
> In environments with high churn rates, stale data can build up in the cache. When running as a pull through cache the Registry periodically removes old content to save disk space. Subsequent requests for removed content causes a remote fetch and local re-caching.
> To ensure best performance and guarantee correctness the Registry cache should be configured to use the `filesystem` driver for storage.

- [x] **已知问题：** 当我们在配置文件开启了 `delete.enabled.true` 那么调度程序会自动清理过期的镜像层或未使用的镜像标签