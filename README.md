# CodeCop.PostSharp.AutoAspectResolver 
Ever wanted to inject dependencies into PostSharp method interception aspects in a seamless way? Meet AutoAspectResolver, just plug-in you DI container of choice and watch PostSharp aspect dependencies being resolved at runtime with the help of <a href="http://getcodecop.com" target="_blank">CodeCop</a>.

# Instructions 

<h3>Create your PostSharp aspects of type MethodInterceptionAspect or OnMethodBoundaryAspect</h3>
Notice that both these example aspects have a constructor dependency that will be automatically resolved at runtime, so just add the dependencies you need.
```
    [Serializable]
    public class MyPostSharpAspect : MethodInterceptionAspectt
    {
        private readonly ILogger _logger;

        public MyPostSharpAspect() { }

        public MyPostSharpAspect(ILogger logger)
        {
            _logger = logger;
        }

        public override void OnInvoke(MethodInterceptionArgs args)
        {
            Console.WriteLine("I am the PostSharp overriding logic!");

            if (_logger != null)
            {
                Console.WriteLine("Logger dependency was injected!");
            }
        }
    }

    [Serializable]
    public class MyPostSharpAspect2 : OnMethodBoundaryAspect
    {
        private readonly ILogger _logger;

        public MyPostSharpAspect2() { }

        public MyPostSharpAspect2(ILogger logger)
        {
            _logger = logger;
        }

        public override void OnEntry(MethodExecutionArgs args)
        {
            Console.WriteLine("I am the PostSharp on entry logic!");

            if (_logger != null)
            {
                Console.WriteLine("Logger dependency was injected!");
            }
        }
    }
```
<h3>Pluging in your DI container</h3>
First make your DI container implement the IAspectContainer interface, this is an example for the <a href="http://autofac.org/" target="_blank" >Autofac</a> container.
```
public class AppContainer : IAspectContainer
    {
        private readonly IContainer _container;

        public AppContainer()
        {
            var builder = new ContainerBuilder();
            
            // Register your app dependencies, including your aspects concrete types
            builder.RegisterType<Logger>().As<ILogger>();
            builder.RegisterType<MyPostSharpAspect>();
            builder.RegisterType<MyPostSharpAspect2>();

            _container = builder.Build();
        }

        // Implement your container resolving logic here
        public object Resolve(Type type)
        {
            return _container.Resolve(type);
        }

       // This interface method is needed to return (not resolve!!!) all types that implement PostSharp's IAspect interface  registered in the container
        public IEnumerable<Type> GetRegisteredAspectTypes()
        {
            return _container.ComponentRegistry.Registrations
                .Where(r => typeof(IAspect).IsAssignableFrom(r.Activator.LimitType))
                .Select(r => r.Activator.LimitType);
        }
    }
```
<h3>Bootstrapping</h3>
You can bootstrap AutoAspectResolver in 2 ways:

1- If you implement the GetRegisteredAspectTypes method on your container you can use the AutoResove method like so:
```
            // Instantiate the container (the one you made implement the IAspectContainer interface)
            var container = new AppContainer();

             // Instantiate AutoAspectResolver passing your container instance
             // Pass the optional CodeCop license parameter or else you will be working in the Free mode
            var autoAspectResolver = new AutoAspectResolver(container);
         
            // Tell AutoAspectResolver to automatically resolve all aspects 
            autoAspectResolver.AutoResolve();
            
            // Nothing more is needed, just start your app logic
            StaticClass.StaticMethod();
            StaticClass.StaticMethod2();
```
2- If you don't implement the GetRegisteredAspectTypes method you should indicate the aspect types you want to auto resolve:
```
            // Instantiate the container (the one you made implement the IAspectContainer interface)
            var container = new AppContainer();
            
            // Instantiate AutoAspectResolver passing your container instance
            // Pass the optional CodeCop license parameter or else you will be working in the Free mode
            var autoAspectResolver = new AutoAspectResolver(container);
         
            // Individualy specify which aspects to auto resolve
            autoAspectResolver.AutoResolve<MyPostSharpAspect>();
            autoAspectResolver.AutoResolve<MyPostSharpAspect2>();

            // Nothing more is needed, just start your app logic
            StaticClass.StaticMethod();
            StaticClass.StaticMethod2();
```

<h3>Grab the nuget package</h3>
AutoAspect resolver is available on nuget, just install the <a href="https://www.nuget.org/packages/CodeCop.PostSharp.AutoAspectResolver/" target="_blank">package</a> and your good to go.
