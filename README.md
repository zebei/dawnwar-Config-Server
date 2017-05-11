# dawnwar-Config-Server
dawnwar by zebei
配置提供服务


指定刷新范围

上面的例子中，我们通过向服务实例请求Spring Cloud Bus的/bus/refresh接口，从而触发总线上其他服务实例的/refresh。但是有些特殊场景下（比如：灰度发布），我们希望可以刷新微服务中某个具体实例的配置。

Spring Cloud Bus对这种场景也有很好的支持：/bus/refresh接口还提供了destination参数，用来定位具体要刷新的应用程序。比如，我们可以请求/bus/refresh?destination=customers:9000，此时总线上的各应用实例会根据destination属性的值来判断是否为自己的实例名，若符合才进行配置刷新，若不符合就忽略该消息。

destination参数除了可以定位具体的实例之外，还可以用来定位具体的服务。定位服务的原理是通过使用Spring的PathMatecher（路径匹配）来实现，比如：/bus/refresh?destination=customers:**，该请求会触发customers服务的所有实例进行刷新。

架构优化

既然Spring Cloud Bus的/bus/refresh接口提供了针对服务和实例进行配置更新的参数，那么我们的架构也相应的可以做出一些调整。在之前的架构中，服务的配置更新需要通过向具体服务中的某个实例发送请求，再触发对整个服务集群的配置更新。虽然能实现功能，但是这样的结果是，我们指定的应用实例就会不同于集群中的其他应用实例，这样会增加集群内部的复杂度，不利于将来的运维工作，比如：我们需要对服务实例进行迁移，那么我们不得不修改Web Hook中的配置等。所以我们要尽可能的让服务集群中的各个节点是对等的。

我们主要做了这些改动：

在Config Server中也引入Spring Cloud Bus，将配置服务端也加入到消息总线中来。
/bus/refresh请求不在发送到具体服务实例上，而是发送给Config Server，并通过destination参数来指定需要更新配置的服务或实例。
通过上面的改动，我们的服务实例就不需要再承担触发配置更新的职责。同时，对于Git的触发等配置都只需要针对Config Server即可，从而简化了集群上的一些维护工作。
