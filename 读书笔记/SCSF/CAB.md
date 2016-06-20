#CAB
Composite ApplicationBlock 复合应用程序块

don't directly reference each other

* module: standalone project to be used in a composite user interface
		  在组合的界面里独立运行的项目  
* shell: the host form for the composite user interface  
		  组合界面的主人  

插件式的架构系统

shell should inherit FormShellApplication<,>
config file: ProfileCatalog.xml
make sure all project build into the same build directory.

The CAB doesn't just call the usual startup code.
inherit ModuleInit  override load

The CAB is doing a lot of work for us when we call that .Run() method on a FormShellApplication class:
* It's instantiationg and showing our Form1(our shell);
* It's checking for an XML file called ProfileCatalog.xml;
* It's loading the assemblies it find listed in there;
* It's then looking for any ModuleInit classes in them, and calling

WorkItems
a run-time container of componets that are collaborating to fulfill a use case
1. The Items collection
2. The Servies collection
3. The WorkItems collection

Dependency Injection
in short it is a means of allowing a main class to use a second, dependent class without directly referencing that class.
Some external entity will provide the second class to the main class at run-time,it will 'inject' the 'dependency'.
Obviously to be useful the second class will need to implement an interface that the main class knows about.

The aim of this is to let us change the behaviour of our class structure by changing which second class is injected into the main class. Because the main class has no hard dependency on the second class this can be done at runtime.

Thus our code is very flexible, easy to modify and loosely coupled.


