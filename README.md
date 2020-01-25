## 关于 LuCI in Docker
在使用docker的过程中，很多容器的配置文件需要管理，使用过程中不少人对命令行及配置文件不熟悉，所以考虑将luci装入容器，配合 [luci-lib-docker](https://github.com/lisaac/luci-lib-docker) 以进行docker容器的配置文件管理。
`luci-in-docker`将`openwrt`中`ubus`去除，宿主为`alpine`，方便后期增加插件。

## 目录结构
```
/
  |- external         # 外部目录，需要外部挂载
    |-cfg.d
      |-config        # UCI cofnig 目录，用于存放配置文件，启动后挂载至/etc/config
    |-plugin          # 插件目录
      |-luci-app-diskman    # 插件，会忽略以 _ 开头的目录，方便调试，插件结构如下：
        |-root        # 插件所需的 root 目录，合并至/tmp/.luci/
        |-luasrc      # 插件所需的 lua 文件目录，合并至/tmp/.luci/usr/lib/lua/luci
        |-htdoc       # 插件所需的 html 文件目录，合并至/tmp/.luci/www
        |-po          # 插件所需的 po 文件目录
        |-depends.lst # 插件所需要 alpine 依赖列表文件, 依赖用' '隔开
        |-init.sh     # 插件所需的初始化脚本
      |-...
  |- internal         # 内部目录，luci-in-docker 自带插件目录
    |-plugin          # 内部插件目录
      |-luci          # luci 目录
        |-root        # 插件所需的 root 目录，合并至/tmp/.luci/
        |-luasrc      # 插件所需的 lua 文件目录，合并至/tmp/.luci/usr/lib/lua/luci
        |-htdoc       # 插件所需的 html 文件目录，合并至/tmp/.luci/www
        |-po          # 插件所需的 po 文件目录
        |-depends.lst # 插件所需要 alpine 依赖列表文件, 依赖用' '隔开
        |-init.sh     # 插件所需的初始化脚本
      |-...
  |tmp
    |-.luci           # 合并后的 luci root 目录
```
- 通过遍历 internal/external 目录下 plugin 中的各个插件目录，将其合并至/temp/.luci目录中，并修改 path 环境变量
- 同时保证兼容性和持久性 config 目录存储位置为external/cfg.d/config, 挂载至/etc/config
- 遍历的同时会安装插件目录下 depends.lst 中需要的依赖，并且执行插件目录下 init.sh

## 运行容器
```
docker pull lisaac/luci
docker run -d \
  --name luci \
  --restart unless-stopped \
  --privileged \
  -p 80:80 \
  -e TZ=Asia/Shanghai \
  -v $HOME/pods/luci:/external:rslave \
  -v /media:/media:rshared \
  -v /dev:/dev:rslave \
  -v /:/host:ro,rshared \
  -v /usr/bin/docker:/usr/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --tmpfs /tmp:exec \
  --tmpfs /run \
  lisaac/luci
```

## 插件
> 插件支持 lua 编写的源码，或者编译完成后 ipk 中的 data 目录中的内容
以添加插件 [luci-app-diskman](https://github.com/lisaac/luci-app-diskman) 为例，软件容器的时候，已经通过
`-v $HOME/pods/luci:/external` 将`$HOME/pods/luci`映射到容器中`/external`，所以我们只需:
```
mkdir -p $HOME/pods/luci/plugin
git clone https://github.com/lisaac/luci-app-diskman $HOME/pods/luci/plugin/luci-app-diskman
```
接下来重启容器即可
> tips: 由于 `luci-app-diskman` 中的依赖较多，第一次启动安装依赖可能会比较慢，需要多等一会，通过 `docker logs luci` 可以看到运行日志


添加插件 [luci-app-dockerman](https://github.com/lisaac/luci-app-dockerman):
```
git clone https://github.com/lisaac/luci-lib-docker $HOME/pods/luci/plugin/luci-lib-docker
git clone https://github.com/lisaac/luci-app-dockerman $HOME/pods/luci/plugin/luci-app-dockerman
docker restart luci
```
