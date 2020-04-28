# SPI 

1. SPI 是一个面向开发者的扩展点，接口上标注了@SPI 代表这是一个面向开发者扩展的接口。

     

2. SPI 可以有多个实现，谁来选择哪个实现呢？

   2.1 我们可以像普通的策略模式一样，根据一个名字选择一个具体的实现来使用。
    2.2   Dubbo 也提供了当调用 SPI 接口的某个方法时，动态根据 URL的某个 key 来动态查找实现进行调用。
            这种动态根据 URL 中的某个 key 来查找实现进行调用 是由一个叫做Adaptive 实现类完成的。
    2.3  SPI 接口需要根据动态查找的实现调用的方法，需要标注@Adaptive 注解。
    2.4  SPI 接口的 Adaptive 实现 可以理解为是一层代理。 代理用来根据 URL 中的某个 key 查找具体的扩展点实现进行调用。
    2.5  SPI 接口的 Adaptive 实现对于没有标注@Adaptive 的方法，是不代理的，直接抛出异常。

   

3. SPI 的 Adaptive 实现如何选择使用哪个 SPI 扩展点实现呢？
     3.1  Dubbo 是以 URL 为基础来选择扩展点的。				
     3.2  选择 URL 上的那个 key 呢？ 这个是有优先级的：
     					a.  方法上指定的@Adaptive(key1,key2) ，先找 key1 如果 url 中有 key1，则使用 key1， 否则使用 key2
                 b.  如果 key1,key2 在 URL 中都没有，则使用@SPI(key3) 中指定的 key3, 
                 c.  如果 key3 也没有，则使用SPI接口名进行转换，如 AbcXyz => abc_xyz



# Adaptive 

这个就像 以上 SPI 注释中说明的一样，@Adaptive 是 用来标注在 某个被@SPI标注的接口 的某个方法上的。

某个方法标注了@Adaptive ，这个方法就可以根据 URL 中的某个key 进行动态查找实现， 由这个动态查找的实现进行完成方法执行。

Adaptive 叫做自适用， 是指某个方法，可以根据 URL 查找特定的 key，根据 key 的 value 找到具体的扩展实现进行调用。

解决了什么问题呢？ 调用者不需要在代码里手写某个 key 了进行查找实现了。这个 key是在接口定义时就确定下来的。而 key 的 value 是在 URL 中的。 是不是很动态~~





# Activate 

这个 Activate 是另一种概念，我来尝试解释一下：
我们知道一个接口，有多个实现，我们可能根据某种场景下选择一个具体的实现进行使用。
但是在一些场景下，我们可能要选择一批都满足条件的实现。 Activate 就是用来干这个的。

应用场景： @Activate 在 Dubbo框架 内部只在 Filter 上有使用。

1. 考虑一个问题，如何选择匹配的实现呢？
	
  - 1.1  在一个 SPI具体实现上标注@Activate(group={xx,yy})
  
  - 1.2  使用
> ExtensionLoader  loader  = ExtensionLoader.getExtensionLoader(Filter.class);
> String key = "service.filter";
> String group = "provider";
> List<Filter> filters = loader.getActivateExtension(invoker.getUrl(), key, group);

>  ExtensionLoader  loader  = ExtensionLoader.getExtensionLoader(Filter.class);
>  String key = "refer.filter";
>  String group = "provider";
>  List<Filter> filters = loader.getActivateExtension(invoker.getUrl(), key, group);

从 url 中取得 key 对应的值，如果值中包括 group 对应的值，则该 Filter 匹配。



2. 考虑一个问题，排序问题。
					有 order 属性 也是支持的。


