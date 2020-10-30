# J2EE设计模式

**J2EE 模式**：这些设计模式特别关注表示层。MVC 模式（MVC Pattern）、业务代表模式（Business Delegate Pattern）、组合实体模式（Composite Entity Pattern）、数据访问对象模式（Data Access Object Pattern）、前端控制器模式（Front Controller Pattern）、拦截过滤器模式（Intercepting Filter Pattern）、服务定位器模式（Service Locator Pattern）、传输对象模式（Transfer Object Pattern）。



[**MVC 模式（MVC Pattern）**](mvc.md)

代表 Model-View-Controller（模型-视图-控制器） 模式。这种模式**用于应用程序的分层开发**。

- **Model（模型）** - 模型代表**一个存取数据的对象或 JAVA POJO**。它也可以带有逻辑，在数据变化时更新控制器。
- **View（视图）** - 视图代表**模型包含的数据的可视化**。
- **Controller（控制器）** - 控制器作用于模型和视图上。它*控制数据流向模型对象，并在数据变化时更新视图*。它使视图与模型分离开。

  

**[业务代表模式（Business Delegate Pattern）](business_delegate.md)**

**用于对表示层和业务层解耦**。它基本上是**用来减少通信或对表示层代码中的业务层代码的远程查询功能**。在业务层中我们有以下实体。

- **客户端（Client）** - 表示层代码可以是 JSP、servlet 或 UI java 代码。
- **业务代表（Business Delegate）** - 一个**为客户端提供的入口类**，它**提供了对业务服务的访问**。
- **查询服务（LookUp Service）** - 查找服务对象负责**获取相关的业务实现对象**，并提供业务代表对象对业务对象的访问。
- **业务服务（Business Service）** - 业务服务接口。实现了该业务服务的实现类，**提供了实际的业务实现逻辑**。

 

[**组合实体模式（Composite Entity Pattern）**](composite_entity.md)

**用在 EJB 持久化机制中**。一个组合实体是一个 EJB 实体 bean，代表了对象的图解。当更新一个组合实体时，内部依赖对象 beans 会自动更新，因为它们是由 EJB 实体 bean 管理的。以下是组合实体 bean 的参与者。

- **组合实体（Composite Entity）** - 主要的实体 bean，可以是粗粒的，或者可以包含一个粗粒度对象，用于持续生命周期。
- **粗粒度对象（Coarse-Grained Object）** - 包含依赖对象，有自己的生命周期，也能管理依赖对象的生命周期。
- **依赖对象（Dependent Object）** - 依赖对象被粗粒度对象依赖，生命周期依赖于粗粒度对象。
- **策略（Strategies）** - 策略表示如何实现组合实体。

 

[**数据访问对象模式（Data Access Object Pattern）或 DAO 模式**](data_access_object .md)

**用于把低级的数据访问 API 或操作从高级的业务服务中分离出来**。以下是数据访问对象模式的参与者。

- **数据访问对象接口（Data Access Object Interface）** - 定义了在一个模型对象上要执行的标准操作。
- **数据访问对象实现类（Data Access Object concrete class）** - 实现了数据访问对象接口，负责从数据源获取数据，数据源可以是数据库，也可以是 xml，或者是其他的存储机制。
- **模型对象/数值对象（Model Object/Value Object）** - 简单的 POJO，包含了 get/set 方法来存储通过使用 DAO 类检索到的数据。

 

[**前端控制器模式（Front Controller Pattern）**](front_controller.md)

**用来提供一个集中的请求处理机制，所有的请求都将由一个单一的处理程序处理**。该处理程序可以做认证/授权/记录日志，或者跟踪请求，然后把请求传给相应的处理程序。以下是这种设计模式的实体。

- **前端控制器（Front Controller）** - **处理应用程序所有类型请求的单个处理程序**，应用程序可以是基于 web 的应用程序，也可以是基于桌面的应用程序。
- **调度器（Dispatcher）** - 前端控制器**使用一个调度器对象来调度请求到相应的具体处理程序**。
- **视图（View）** - 视图是为请求而创建的对象。

 

[**拦截过滤器模式（Intercepting Filter Pattern）**](intercepting_filter.md)

**用于对应用程序的请求或响应做一些预处理/后处理**。定义过滤器，并在把请求传给实际目标应用程序之前应用在请求上。**过滤器可以做认证/授权/记录日志，或者跟踪请求，然后把请求传给相应的处理程序**。以下是这种设计模式的实体。

- **过滤器（Filter）** - 过滤器在请求处理程序执行请求之前或之后执行某些任务。
- **过滤器链（Filter Chain）** - 过滤器链带有多个过滤器，并在 Target 上按照定义的顺序执行这些过滤器。
- **Target** - 是请求处理程序。
- **过滤管理器（Filter Manager）** - 管理过滤器和过滤器链。
- **客户端（Client）** - Client 是向 Target 对象发送请求的对象。

 

[**服务定位器模式（Service Locator Pattern）**](service_locator.md)

**用在我们想使用 JNDI（Java Naming and Directory Interface） 查询定位各种服务的时候**。**考虑到为某个服务查找 JNDI 的代价很高，服务定位器模式充分利用了缓存技术**。在首次请求某个服务时，服务定位器在 JNDI 中查找服务，并缓存该服务对象。当再次请求相同的服务时，服务定位器会在它的缓存中查找，这样可以在很大程度上提高应用程序的性能。以下是这种设计模式的实体。

- **服务（Service）** - 实际处理请求的服务。对这种服务的引用可以在 JNDI 服务中查找到。
- **Context / 初始的 Context** - JNDI Context 带有对要查找的服务的引用。
- **服务定位器（Service Locator）** - 服务定位器是通过 JNDI 查找和缓存服务来获取服务的单点接触。
- **缓存（Cache）** - 缓存存储服务的引用，以便复用它们。
- **客户端（Client）** - Client 是通过 ServiceLocator 调用服务的对象。

 

[**传输对象模式（Transfer Object Pattern）**](transfer_object.md)

**传输对象模式（Transfer Object Pattern）**：**用于从客户端向服务器一次性传递带有多个属性的数据**。*传输对象也被称为数值对象，传输对象是一个具有 getter/setter 方法的简单的 POJO 类，它是可序列化的，所以它可以通过网络传输，它没有任何的行为*。服务器端的业务类通常从数据库读取数据，然后填充 POJO，并把它发送到客户端或按值传递它。对于客户端，传输对象是只读的。客户端可以创建自己的传输对象，并把它传递给服务器，以便一次性更新数据库中的数值。以下是这种设计模式的实体。。

- **业务对象（Business Object）** - 为传输对象填充数据的业务服务。
- **传输对象（Transfer Object）** - 简单的 POJO，只有设置/获取属性的方法。
- **客户端（Client）** - 客户端可以发送请求或者发送传输对象到业务对象。



参考：https://www.runoob.com/design-pattern/design-pattern-intro.html
