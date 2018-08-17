# kendoDDD
基于Golang的DDD实践

分享一点不成熟的理解，还请本着交流进步的大原则喷之。从去年开始接触和套用DDD以来，已经有1年多时间了。也先后在2个生产项目中主导应用，都是基于.Net Core的，完全参考https://github.com/EduardoPires/EquinoxProject 该项目搭建的基础框架。

## 一、一些概念

　　DDD经典分层：
![](https://github.com/KendoCross/kendoDDD/blob/master/assets/01.png)
 分层架构的一个重要原则是：每层只能与位于其下方的层发生耦合。严格分层架构，某层只能与直接位于其下方的层发生耦合；松散分层架构，则允许任意上方层与任意下方层发生耦合。大原则如此，我一般都是采用松散分层，严格的太夸张，在团队里推广起来挺难的。

　　CQRS：

　　命令查询职责分离，是由Betrand Meyer（Eiffel语言之父，OCP提出者）提出来的。命令(Command):不返回任何结果(void)，但会改变对象的状态。查询(Query):返回结果，但是不会改变对象的状态，对系统没有副作用。在我的实践过程中，其实还是让命令返回了一些主键之类的。

　　ES事件溯源：

　　在CQRS中，每一个确定的命令操作，不论成功还是失败，只要执行之后就产生相应的事件（Event）。这样只需要把所有系统运行的Event，以及其产生时的数据记录起来，这就是系统的历史记录了，并且能够方便的回滚到某一历史状态。Event Sourcing就是用来进行存储和管理事件的。

　　粗略的知道了这几个概念之后，基本上就可以用来理解和构建最基础的领域驱动架构了。

## 二、一般实践

　　总体分层结构如下：
![](https://github.com/KendoCross/kendoDDD/blob/master/assets/02.png)
　　1.presentation 表现层。Web框架选择了BeeGo，Beego是基于MVC的。我将V和C层都放在了表现层。路由设置、Views、以及Controllers这些统在一起都算是表现层的。领域驱动设计，更多关注的是业务，以及其变更时能够比较好的扩展与维护。表现层逻辑也很简单，主要是承接Beego框架转发过来的请求，然后通过memoryBus将请求发布出去，至于是谁订阅了该请求，统一委托给 第三方 HttpServer来处理，这也是 迪米特法则的精要。解除了表现层或者说web请求与具体的处理该请求的业务服务的直接关联（耦合）。这样做的好处，只举一例。业务层可以是独立的分布式服务，不一定跟表现层在一个进程服务里。

　![](https://github.com/KendoCross/kendoDDD/blob/master/assets/03.png)
 ![](https://github.com/KendoCross/kendoDDD/blob/master/assets/04.png)

　　2.application应用层。DDD里应用层是很薄的一层，只作为计算机领域到业务领域的过渡层。比如计算机能够识别和传输的肯定是2进制字节流，这一层可以充当翻译，把这些晦涩难懂的机器“语言”，转化为领域业务人员建模出来的语言。或者说是高级计算机编程语言，这里一般会有专门的ViewMode来承接所需的参数数据。这一层直接消费领域层，并且开始记录一些系统型功能，比如日志、事件溯源。
![](https://github.com/KendoCross/kendoDDD/blob/master/assets/05.png)

　　3. domain领域层。领域驱动设计里最核心的部分了，可以细拆分为聚合根、实体，领域服务等一大堆其他概念。这里不展开详细说明了，简单的理解下 聚合根，负责整个聚合业务的所有功能就行了。这里已Protocol协议为例，该类直接负责完与协议相关的所有业务，对内各种封装，完成所有所需的功能，由聚合根对外统一提供方法。

　　4. infrastructure基础架构层。这一层一般主要将所有公共功能抽象整理出来，作为独立的对外输出一些帮助类等。但我这里只是重点放置了仓储功能，也就是和数据库打交道的功能。这一层也是讲求和业务逻辑无关，只重点提供通用的功能。

　　5.crossutting、ddd_interfaces、dddcore，这些都不是DDD经典分层里的，主要是用来方便实现DDD所增加和分离出来的一些接口、和基础概念实现。

## 三、一点感悟

　　对DDD的理解和应用其实还很不成熟，只是对过去的一点总结。Golang 也是新手，只是看了一遍《Go语言圣经》以及astaxie大神的使用Golang构建web应用。

　　确实很Go的面向对象设计，没有继承，而是采用组合的方式来实现OO里的“继承”。尽可能的使用组合而不是继承，也算是面向对象设计的一个原则，Golang天然就直接遵循了。“封装”也很有意思，直接通过首字母是否大小写，“多态”还没怎么了解透彻就不可描述了。再使用Golang实现DDD的时候，也发现了如果有“泛型”就好了的感概。这个时候更能深刻的理解和体会，泛型的好处。不得不佩服下C#的设计者，很早就有了泛型。

　　简单的构造出来了一个DDD的实现，不成熟的地方还有很多，但是迭代是个好东西，慢慢的理解和完善。
