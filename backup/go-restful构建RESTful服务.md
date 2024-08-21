# RESTful 理念

REST是Representational State Transfer的缩写，即表现成状态转换。这种软件风格，通过基于HTTP的一组约束和属性，提供网络服务，主要特点包括：
- 统一接口（Uniform Interface）,资源通过一致、可预见的URI及请求方法来操作；
- 无状态（Stateless），请求状态由客户端维护，服务端不做保存；
- 可缓存（Cacheable），可以通过缓存提高系统性能；
- 分层系统（Layered System）；
- 资源的表示（Resources Representation）：资源可以有多种表示形式（如JSON、XML），客户端通过这些表示形式来与服务器交互。客户端和服务器之间通过这些表示形式来传递数据。

统一接口 是 RESTful 服务最大的特点。统一接口的核心思想是，将服务抽象成各种 资源 ，并通过一套一致、可预见的 URI 以及 请求方法（ Request Method ）来操作这些资源。这样一来，掌握了一种资源的使用方法，便可延伸到其他资源上，达到举一反三的效果。
