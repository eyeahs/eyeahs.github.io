---
published: false
---

### Bindings in the graph

위 예제는 어떻게 좀 더 일반적인 바인딩들을 가진 component를 만드는지를 보여준다. 하지만 그래프에  
The example above shows how to construct a component with some of the more typcial bindings, but there are a variety of mechanisms for contributing bindings to the graph. The following are available as dependencies and may be used to generate a well-formed component:

* Those declared by @Provides methods within a @Module referenced directly by @Component.modules or transitively via @Module.includes
* Any type with an @Inject constructor that is unscoped or has a @Scope annotation that matches one of the component’s scopes
* The component provision methods of the component dependencies
* The component itself
* Unqualified builders for any included subcomponent
* Provider or Lazy wrappers for any of the above bindings
* A Provider of a Lazy of any of the above bindings (e.g., Provider<Lazy<CoffeeMaker>>)
* A MembersInjector for any type
