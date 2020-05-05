## Go项目实践

### 大概想法

#### 工具作用

爬虫，美剧更新
大致想法，
追老剧
每天去爬相关网站一次（提前配置），然后读取自己加入的美剧名词（中，英多个），然后保留上次的版本号，然后如果看完了，需要把版本号加1，如果一段时间没看，那么自然一本有多条

搞个db，存信息，增删改查
gorm 做db
angular/react 前端
service/ 
api gateway， gin/mux?
docker
k8s
用gcp来部署
监控？
tracking？
logging？

api
sites CRUD
tv CRUD
users CRUD
look for tv updates

JOB
update tv episode to -> updateItems



### 详细设计



### 具体实践

* api 

  * 查看网站
  * 查看tv
  * 查看更新历史
  * 添加tv
  * 添加网站

* 数据表

  * tv，包含 zh_name, en_name, season, episode, desc, UserId, createAt, updateAt,
  * sites, 包含 Name, url, UserId, status,createAt, updateAt,
  * update_items, 包含 tv_id, site_id, season, episode, url, UserId, createAt, updateAt,

* repository

  * tvs
    * insert
    * update
    * findByZhName, findByEnName, findAll
    * delete
  * sites
    * insert
    * update
    * findByName, findAll
    * delete
  * updateItems
    * insert
    * delete
    * update
    * findByTv

* services

  * findAllTv
  * findUpdateItemsByTv
  * addTv

* api

  * findAllTv

  * findUpdateItemsByTv

  * addTv

    

* job

  * find tv updates from each sites, insert into updateitems

* UserManagement

  * User
  * Role

### 实现过程

1. 下载了github的项目进行修改，gorm, user/role 框架，dto/service/controller等
2. 修改需要的model，api等，先简单后复杂
3. 配置docker, docker-compose, 启动运行，数据库连接 pgadmin使用
4. 修改seed服务，目前没数据， 
5. 编写job，测试自动运行(random time)
6. 修改测试api
7. test 要写吧？？？？
8. 部署google cloud
9. 配置域名

#### 碰到的问题

* no package in GOROOT, 解决发现是go.mod中本项目的module名字有问题，导致项目中的package name和这个不符，本地的几个package不能被build
* cgo.a: permission denied, 
* docker 文件编写，还要写docker-compose 
  * docker 运行, docker-compose up 作用? https://blog.csdn.net/u011781521/article/details/80464826
  * docker-compose up 超时, export COMPOSE_HTTP_TIMEOUT=300 https://github.com/docker/compose/issues/3851
  * 启动 pgadmin4 docker run -p 5050:80 -e "PGADMIN_DEFAULT_EMAIL=live@admin.com" -e "PGADMIN_DEFAULT_PASSWORD=password" -d dpage/pgadmin4
  * 

### TODO 


查新剧
1. 分数
2. tag（剧情，法律，医务等）

想做一个类似bookin的网站，用户可以注册，可以得到自己关注的剧的更新


电商
医疗
互联网
怎么融合互联网，机房，容灾，算法(algortihm, data structure)
grab health, search good doctor