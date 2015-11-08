# CodeCop.PostSharp.AutoAspectResolver
Ever wanted to inject dependencies into PostSharp method interception aspects in a clean and easy way? Meet AutoAspectResolver, just plug-in you DI container of choice and watch PostSharp aspect dependencies being resolved at runtime with the help of CodeCop.

# Instructions
<h3>Pluging in your DI container:</h3>
First make your DI container implement the IAspectContainer interface, this is an example for the <a href="#" target="_blank" >Autofac</a>
```
var code = File.ReadAllText("PathToYourClassFile");
```
