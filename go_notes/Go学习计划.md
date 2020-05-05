java程序员学Go

语言学习，无非是学习语法，然后写，我也曾经用go工作了一段时间，在同事前人基础的代码架构上(microservice，httprest)写，但是总是感觉不顺手，一是不容易记住，当我换了工作重写java之后，在想捡起go不是那么容易的一件事，原因是什么？甚至我觉得我写shell，都没有那么容易忘，虽然一年也写不了几个shell，我大概想了几个原因
1. 年纪大了，记不住
2. 没把Go放在心上，没有深入去理解，接受他
3. 没有更多的阅读开源代码，熟悉Go的使用（特别是指针，对象，接口的应用，Go很有一套）
4. 没有更多自己放手实践的机会

正好趁现在自己无业的时间，捡起来，把上面几个补上，同时下一份工作如果顺利的话，大概率自己要去搞infra，在k8s和docker里写些基础通用的架构，提前准备一下。不打无把握的仗。

接下来学习的一个方向:
1. 更多的理解Go的原理, 包括底层数据结构，GC逻辑，语法优化，特别是Go的高级特性
2. 阅读Go本身的库
3. 理解k8s，docker从developer角度，然后尝试看源码
4. 写一些代码，目前还没啥想法，但是后面会有的

也许更好的方式是，从自己理解java的角度去理解Go。


语法解析， https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-typecheck/


高级特性， 反射使用，接口（多态），指针，异常处理(defer,recover), goroutine相关， channel，
api熟悉程度

k8s https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/ , kubernetes up and running (pdf), 
https://www.infoq.cn/article/te70FlSyxhltL1Cr7gzM

docker https://docs.docker.com/docker-for-mac/
源码，实践，本地

写什么？
1. 爬虫，拉取自己关注的美剧，更新链接！
2. 爬虫，hattrick，转会列表自动关注

其他？？

资料
https://golang.org/
go weekly
https://studygolang.com/
twitter,
fb
github?
后面再补充。。。

go 操作数据库

https://andrewpillar.com/programming/2020/04/07/working-with-sql-relations-in-go-part-1/

go restapi
https://medium.com/@hagenverfolgt/build-a-rest-api-in-golang-with-swagger-and-hot-reload-of-everything-6247a8ae8618