
- Module: where Daggger looks for a proper way to instantiate the object. Tells it HOW to create the object

- Component: is the bridge between module and the classes that need it. As you cant use the module directly. Component takes the modules and gives the dependency back to the dependent classes. If a component has multiple modules that are dependent on others, they are declared in here


Essentially it is the factory. Can use it without a module, as long as the class has a default constructor with @inject
This tells dagger that when we request an object from Dagger, it calls a constructor to provide the object.


- Note that it doesnt have to be the default constructor with no arguments. Can also have something like this where the constuctor has an argument. BUT this argument must have a constructor with @inject so Dagger knows

- *Provides* are concrete methods that tell Dagger how to create the object. Unlike binds, Provides methods can actually create the object and return. Binds methods can only be abstract methods

```java
 @Inject
    CommandRouter(HelloWorldCommand helloWorldCommand) {
        commands.put(helloWorldCommand.key(), helloWorldCommand);
    }
  ```

```java
//to create the component
AppComponent appComponent = DaggerAppComponent.create()
```

- What happens if you want to inject in an interface? You cant have an @inject constructor on an interface. But you can use the ```@Binds``` method in the module

```java
@Module
abstract public class HelloWorldModule {
    @Binds
    abstract Command helloWorldCommand(HelloWorldCommand command);
}
```


  - When creating the component, if a module needs to have values passed in, ie context, then you need to create AND pass in the module , rather than letting the system create one automatically ie

    ```java
    DaggerAppComponent.builder
    ```


- Dependent classes must be declared in the component, so that Dagger can use the public variables and initialize them

- ```java
  @Component(modules = CoffeeProvider.class)
  public interface CoffeeComponent {
  
      void provideCoffee(CofferUser user); //note doesnt have to be called inject
  }
  ```

this means that the CofferUser class will have any provides instances filled by the associated module

- remember that the component is required to initialize the dependency. Without the component, this cannot happen

- Can also provide a method to get the dependency directly , and not throught the injection into the main class via 

  ```java
  @Component(modules = CoffeeProvider.class)
  public interface CoffeeComponent {
      
      CoffeeHelper getCoffeeHelper();
  
  }
  ```

- Method injection: 

  ```java
  @Inject
  public void setCoffeeUser(CoffeeHelper coffeeHelper){
  
      this.coffeeHelper = coffeeHelper;
  }
  ```

As soon as the injection occurs, methods with 'inject' will be called with the injected coffeehelper passed in. I guess you do this instead of field injection if there is a bunch of stuff you need to do on the dependency



- Constructor injection: Lets assume we have a class that has two arguments to its constructor.
  If a module can **provide** the two arguments, then that module can also provide the instance of the class via constructor injection. To make this happen you MUST add inject annotaion to the constructor

- ```java
   @Inject
      public CoffeeHelper(int water, Flavor flavor) {
          this.water = water;
          this.flavor = flavor;
      }
   ```
  

- We can then have a CoffeeHelper dependency by either fieldInjection or method injection. So basically the INJECT in the constructor tells dagger **make this available in the module**

- Instead of the default constructor injection, can make things more explicit and add

  ``` java 
  @Provides
  public CoffeeHelper CoffeeHelper(int quantity, Flavor flavor){
     return new CoffeeHelper(quantity, flavor);
  }
  ```
  

to the module. This is essentially doing the same thing as the constructor injection, but you can get more hands on with the constructor creation here.

- Note that the provide method has arguments, which MUST be provided by either this module or a dependency module


- Singleton. If it is placed before a method which provides  a dependency, Dagger will create a singleton during init. of the COMPONENT. So if other components are created, they will have a different singleton.
- Singleton can be used on the class declaration of type that has an inject constructor OR on a binds/provides method
  - if one provide method has scope annotation, then the component must have the same annotation
  - can also use the concept of 'scopes', where Singletons exist while a corresponding scope exists ie
    - a global scope (used across the whole application)
    - a local scope (which is used across a few activities)
  - The way this is done is via the components. ie a component created at the application level, a component created at multiple activity level, a component created for a specific screen
  
https://miro.medium.com/max/954/1*TNcwU_fri3QC_s58yXdpxw.png


### Scoping

- allows you to 'preserve' the object instance for the life of the scope
- the issue is defining a relationship between the different scopes, as the ones at the bottom need to
still have the scope of the ones higher up

- scoping comes down to proper use of Components
 - there are two ways to use components
  - Subcomponents
  - Components dependencies

#### Component Dependency

- a component can establish a parent-child dependency between itself and other components.

Suupose we have an Activity with an associated Presenter. ie WarriorActivity and WarriorPresenter
The presenter is only designed for the warrior activity, so we can scope it as @WarriorScreenScope
