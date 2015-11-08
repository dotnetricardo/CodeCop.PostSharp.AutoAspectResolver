# CodeCop.PostSharp.AutoAspectResolver
Ever wanted to inject dependencies into PostSharp method interception aspects in a clean and easy way? Meet AutoAspectResolver, just plug-in you DI container of choice and watch PostSharp aspect dependencies being resolved at runtime with the help of CodeCop.

# Instructions
<h3>Pluging in your DI container:</h3>
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

        // Just implement your container resolving logic here
        public object Resolve(Type type)
        {
            return _container.Resolve(type);
        }

       // This interface method is needed to return (not resolve!!!) all types that implement PostSharp's IAspect interface
        public IEnumerable<Type> GetRegisteredAspectTypes()
        {
            return _container.ComponentRegistry.Registrations
                .Where(r => typeof(IAspect).IsAssignableFrom(r.Activator.LimitType))
                .Select(r => r.Activator.LimitType);
        }
    }
```
