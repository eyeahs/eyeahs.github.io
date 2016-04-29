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
* 
## Reusable scope

Sometimes you want to limit the number of times an @Inject-constructed class is instantiated or a @Provides method is called, but you don’t need to guarantee that the exact same instance is used during the lifetime of any particular component or subcomponent. This can be useful in environments such as Android, where allocations can be expensive.

For these bindings, you can apply @Reusable scope. @Reusable-scoped bindings, unlike other scopes, are not associated with any single component; instead, each component that actually uses the binding will cache the returned or instantiated object.

That means that if you install a module with a @Reusable binding in a component, but only a subcomponent actually uses the binding, then only that subcomponent will cache the binding’s object. If two subcomponents that do not share an ancestor each use the binding, each of them will cache its own object. If a component’s ancestor has already cached the object, the subcomponent will reuse it.

There is no guarantee that the component will call the binding only once, so applying @Reusable to bindings that return mutable objects, or objects where it’s important to refer to the same instance, is dangerous. It’s safe to use @Reusable for immutable objects that you would leave unscoped if you didn’t care how many times they were allocated.

  @Reusable // It doesn't matter how many scoopers we use, but don't waste them.
  class CoffeeScooper {
    @Inject CoffeeScooper() {}
  }

  @Module
  class CashRegisterModule {
    @Provides
    @Reusable // DON'T DO THIS! You do care which register you put your cash in.
              // Use a specific scope instead.
    static CashRegister badIdeaCashRegister() {
      return new CashRegister();
    }
  }


  @Reusable // DON'T DO THIS! You really do want a new filter each time, so this
            // should be unscoped.
  class CoffeeFilter {
    @Inject CoffeeFilter() {}
  }
