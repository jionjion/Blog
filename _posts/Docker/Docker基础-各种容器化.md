# calibre 容器化

安装

`docker pull docker.io/johngong/calibre-web`

启动,并部署.. windows 环境下

`docker run -d --name=my-library -p 8083:8083 -v /V/data/calibre/config:/config -v /V/data/calibre/library:/library johngong/calibre-web`

默认用户名 `admin`  密码 `admin123`

