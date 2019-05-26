> 参考文章：[http://svip.iocoder.cn/Spring/IoC-load-Resource/](http://svip.iocoder.cn/Spring/IoC-load-Resource/ "IoC-load-Resource")


## 1. 统一资源：Resource ##

![子类结构图](Resource.jpg)

- `InputStreamResource`类中的`#getInputStream`方法只允许执行一次，意味着该类只允许读取输入流一次，该类中有一个`boolean`类型的变量`read`，`getInputStream`第一次执行时会设置为`true`，之后再执行会抛出异常。

```java
	/**
	 * This implementation throws IllegalStateException if attempting to
	 * read the underlying stream multiple times.
	 */
	@Override
	public InputStream getInputStream() throws IOException, IllegalStateException {
		if (this.read) {
			throw new IllegalStateException("InputStream has already been read - " +
					"do not use InputStreamResource if a stream needs to be read multiple times");
		}
		this.read = true;
		return this.inputStream;
	}

```

在`ClassUtils`类的`#getDefaultClassLoader`方法中看到Juergen Hoeller写有这样的注释：
> // Cannot access system ClassLoader - oh well, maybe the caller can live with null...

可见Juergen Hoeller也是一个非常幽默的人

## 2. 资源加载器：ResourceLoader
- org.springframework.core.io.ProtocolResolver ，用户自定义协议资源解决策略，作为 DefaultResourceLoader 的 SPI：它允许用户自定义资源加载协议，而不需要继承 ResourceLoader 的子类。
- `ResourcePatternResolver`接口是`ResourceLoader`接口的扩展，可以根据资源的路径通配符匹配并返回多个`Resource`。我们平时使用`classpath*:/config.properties`加载配置文件的时候就是通过这个接口来获取配置文件资源的。其实这个接口的名字会让人产生歧义，看起来像是资源路径模式的处理器而不是资源加载器，换成`PatternResourceLoader`是不是更好呢？
- 最常用的`ResourcePatternResolver`是其子类`org.springframework.core.io.support.PathMatchingResourcePatternResolver`，它除了支持 ResourceLoader 和 ResourcePatternResolver 新增的 "classpath\*:" 前缀外，还支持 Ant 风格的路径匹配模式（类似于 "\*\*/*.xml"）。