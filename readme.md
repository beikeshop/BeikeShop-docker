### 使用说明
1. 在本地或者服务器安装`docker`以及`compose`, https://docs.docker.com/engine/install/
1. 执行 `git clone git@gitee.com:beikeshop/docker.git`
1. 在当前目录创建新文件夹`www`作为网站目录, `mkdir www`
1. 进入docker目录基于模板文件创建配置文件, `cp env.example .env`
1. 根据需要修改`.env`以及`docker-compose`, 然后执行`docker compose up -d`