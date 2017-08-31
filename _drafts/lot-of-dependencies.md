post: why lot of dependecies is evil


Some developers use particular coding style -- they have concept of controller, service and repository. Service usually incapsulates logic 
related to some part of domain. And it contains many methods. Developers do some attempt to follow SOLID principles but in poor way. For instance,
they try to apply single responsibility. To do that they 'delegate' all irrelevant work to the dependencies of the service. 

problems
1. complicated setup in unit tests. 