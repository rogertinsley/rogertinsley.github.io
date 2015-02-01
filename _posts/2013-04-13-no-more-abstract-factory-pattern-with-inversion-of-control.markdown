---
layout: post
title: "No more abstract factory pattern with inversion of control"
date: 2013-04-13 14:48
comments: true
categories: [.NET, C-Sharp]
---

Are you using an inversion of control container with the factory pattern to create objects at run time? 

You might want to read on to see how to write cleaner code...

<!-- more --> 

We have been using [Unity](http://msdn.microsoft.com/en-us/library/hh237493.aspx) lately at work and I have came across the factory pattern sprinkled around the codebase. Its main usage is due to the polymorphic behaviour of the code, that is, we don't know what objects we want to create until run-time.

Say if I describe three different types of car as follows:


``` c# Interface ICar and three types of Cars
public interface ICar
{
    string CarName { get; }
}
 
public class Ford : ICar
{
    public string CarName
    {    
      get { return "Ford"; }
    }
}
 
public class Volkswagen : ICar
{
    public string CarName
    {
        get { return "Volkswagen"; }
    }    
}
 
public class Peugeot : ICar
{
    public string CarName
    {
        get { return "Peugeot"; }  
    }
}
```

Ok, so we'll need a factory to create the cars because we don't know what tyoe of ICar we want until run-time.

``` c# ICar factory
public class CarFactory : ICarFactory
{
    private readonly IUnityContainer unityContainer;
 
    public CarFactory(IUnityContainer unityContainer)
    {
        this.unityContainer = unityContainer;
    }
 
    public ICar Create(string carName)
    {
        if (carName == "Ford") return unityContainer.Resolve<ICar>("Ford");
        if (carName == "Volkswagen") return unityContainer.Resolve<ICar>("Volkswagen");
        if (carName == "Peugeot") return unityContainer.Resolve<ICar>("Peugeot");
 
        throw new NotSupportedException();
    }
}
```

Then I would normally inject the factory into the class via the constructor.

``` c# inject factory through the constructor
private readonly ICarFactory carFactory;
 
public HomeController(ICarFactory carFactory)
{
    this.carFactory = carFactory;
}
```

And to finish up, we'll need to wire up all dependencies using a Unity bootstrapper.

``` c# Unity bootstrapper
public static class Bootstrapper
{
    public static void Initialise()
    {
        var container = BuildUnityContainer();
 
        DependencyResolver.SetResolver(new UnityDependencyResolver(container));
    }
 
    private static IUnityContainer BuildUnityContainer()
    {
        var container = new UnityContainer();  
 
        container.RegisterType<ICarFactory, CarFactory>();
 
        container.RegisterType<ICar, Ford>("Ford");
        container.RegisterType<ICar, Volkswagen>("Volkswagen");
        container.RegisterType<ICar, Peugeot>("Peugeot");
 
        return container;
    }
}
```

Basically, we have created a factory that creates the Car based on some run-time parameter being set, say a drop down list being selected in a web application, or some data that is passed to a WCF service call, etc.

How about refactoring out the factory? I'll start by creating the following constructor.

``` c# Funcy constructor
private readonly Func<string, ICar> carFactory;
 
public HomeController(Func<string, ICar> carFactory)
{
    this.carFactory = carFactory;
}
```

In the bootstrapper I tell Unity how to create the Car objects. That is, by using a function that resolves the ICar based on a name.

``` c# Register the func with Unity
container.RegisterType<Func<string, ICar>>(new InjectionFactory(
    ctx => new Func<string, ICar>(name => container.Resolve<ICar>(name))));
}
```

There is no need for the factory class now, yeah! When we want to create an ICar we'll use the Invoke method of the Func.

``` c# Home controller Invoking the function
public class HomeController : Controller
{
    private readonly Func<string, ICar> carFactory;
 
    public HomeController(Func<string, ICar> carFactory)
    {
        this.carFactory = carFactory;
    }
 
    public ActionResult Create(IndexViewModel indexViewModel)
    {
        ICar car = carFactory.Invoke(indexViewModel.SelectedId);
 
        return new ContentResult
            {
                Content = car.CarName,
            };
    }
}
```

To me, this is cleaner code, less code == win. I've created a sample ASP.NET MVC application using this approach, [take a look](http://github.com/rogertinsley/NoMoreFactories).