CloneExtensions
===============

Clone extension method library. Performs fast, deep clone using simple assignment operations generated by Expression Tree runtime code compilation.

Cloning objects does not sound like a really difficult task. However, making it right is not as easy as it seems to be. There are couple popular methods, including ICloneable interface implementation, copying constructors, Serialization and Reflection. All of them have some pros and cons, but non of them is perfect.

CloneExtensions uses both Reflection and Expression Trees to generate and compile all necessary code at runtime when you use GetCode for the first time. Starting from second time GetClone is as fast as if you'd written you all by yourself.

It's compiled as Portable Class Library and can be used with .NET Framework 4 and higher, Silverlight 5, Windows Phone 8, Windows Store app (Windows 8) and higher.
# Why should I use CloneExtensions? #

Are you sick of writing copy constructors or Clone methods with code like that:
```
public Foo Clone()
{
    var newInstance = new Foo();

    newInstance.Property1 = this.Property1;
    newInstance.Property2 = this.Property2.Clone();
    // ...
    newInstance.Property12 = this.Property12;

    return newInstance;
}
```
Do you need something faster than solutions based on reflection or serialization?

If the answer is yes, CloneExtensions is right for you! Wouldn't it be easier to write just:
```
var newInstance = CloneExtensions.CloneFactory.GetClone(source);
```
or even simpler, using Extension Method:
```
var newInstance = source.GetClone();
```
and get all these assignments out of the box, without implementing Clone method by yourself?
# How does it work? #

Using the power of Expression Trees, introduced in .NET 3.5, you can generate code, compile it into a delegate and run without performance loss. That's exactly how CloneExtensions works. When GetClone method is used for the first time for given parameter type, proper clone expression tree is generated, compiled and remembered. It's done within class static constructor, so runtime takes care to only do that once. After that, whenever you call the method again, compiled delegate is invoke and you get performance as good as if you'd written the clone method code by yourself.
## What can be cloned? ##
- Primitive (`int`, `uint`, `byte`, `double`, `char`, etc.), known immutable types (`DateTime`, `TimeSpan`, `String`) and delegates (including `Action<T>`, `Func<T1, TResult>`, etc)
- `Nullable<T>`
- `T[]` arrays
- Custom classes and structs, including generic classes and structs.

## Following class/struct members are cloned internally##
- Values of public, not readonly fields
- Values of public properties with both get and set accessors
- Collection items for types implementing `ICollection<T>`

## Execution customization ##

You can affect compiled delegate execution providing parameters:
- `CloningFlags` bit flag enum, to specify which members should be copied (fields, properties, collection items, etc.)
- Initializers dictionary, to specify custom initialization code which will be used instead of parameterless constructor call

# Sample usage #

## Simple cloning ##
```
var source = new GenericClass<int> { _field = 10, Property = 10 };
var newInstance = CloneExtensions.CloneFactory.GetClone(source);
```

Simple cloning, using Extension Method syntax
```
var source = new GenericClass<int> { _field = 10, Property = 10 };
var newInstance = source.GetClone();
```

Cloning with initializer specified
```
var source = (AbstractClass)new DerivedClass() { AbstractProperty = 10 };
var initializers = new Dictionary<Type, Func<object, object>>() {
    { typeof(AbstractClass), (s) => new DerivedClass() }
};
var clone = source.GetClone(initializers);
```

# Limitations #

Of course, nothing is perfect. There are couple limitations you have to be aware of, when it comes to GetClone of your custom classes or structs:
- Ony public fields without readonly specified can be cloned
- Only public property with both get and set accessors can be cloned
- Parameterless constructor has to be available or custom initializer has to be provided to create new instance of cloned typed

# Performance #

When it comes to performance it's always easier to compare solutions with each other then describe one of them separately.

Is CloneExtensions the fastest way to clone objects? No, it's not. If you really care about every tick you should write all Clone methods by yourself. That's the fastest way.

Is CloneExtensions faster than other known and popular solutions? It depends on how many instances of the same type you clone. That's because just before first GetClone<T> is performed for the first time Expression Tree has to be generated and cloning method has to be compiled. And to do that CloneExtensions uses Reflection. However, starting from the second time method is used with the same T, it has the same performance as if you'd write the logic by yourself. That's why CloneExtensions is definitely faster then reflection-based solutions and is faster then serialization-based solutions when you clone a lot of instances.
Internals