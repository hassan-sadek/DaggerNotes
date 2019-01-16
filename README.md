
- Module: where Daggger looks for a proper way to instantiate the object. Tells it HOW to create the object

- Component: is the bridge between module and the classes that need it. As you cant use the module directly. Component takes the modules and gives the dependency back to the dependenct classes. If a component has multiple modules that are dependent on others, they are declared in here

  - When createing the component, if a module needs to have values passed in, ie context, then you need to create AND pass in the module , rather than letting the system create one automatically ie

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
  - if one provide method has scope annotation, then the component must have the same annotation
  - can also use the concept of 'scopes', where Singletons exist while a corresponding scope exists ie
    - a global scope (used across the whole application)
    - a local scope (which is used across a few activities)
  - The way this is done is via the components. ie a component created at the application level, a component created at multiple activity level, a component created for a specific screen

