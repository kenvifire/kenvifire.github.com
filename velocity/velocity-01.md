---
layout: page
title: Velocity--Demo1
tagline: Velocity
---
###Velocity执行流程
Velocity 的使用很简单，主要遵循以下几个步骤：

* 初始化Velocity引擎
* 创建一个Context对象
* 把你要渲染的对象放入到Context里
* 选择一个模版文件
* “合并”你的模版文件和数据，产生输出文件

###简单示例
<pre><code>
		Velocity.init();
		VelocityContext context = new VelocityContext();
		context.put("name", "kenvi");
		Template template = null;
		
		try{
			template = Velocity.getTemplate("test.vm");
			
		}catch(ResourceNotFoundException e){
			System.out.println("file not found");
		}catch(ParseErrorException e){
			System.out.println("parsing error");
		}catch(MethodInvocationException e){
			System.out.println("method invoke error");
		}catch(Exception e){
			System.out.println("error");
		}
		
		StringWriter sw = new StringWriter();
		template.merge(context, sw);
		System.out.println(sw.toString());
</code></pre>

###关于模版的位置
关于上面的代码有一个问题需要注意，就是test.vm文件要和生成的class文件放在同一路径下面，否则运行的时候，会报文件找不到的错误，这个原因是因为如果使用默认配置的话，Velocity加载文件模版的FileResourceLoader会在当前路径（也即是`.`）下面查找该文件。

如果需要添加自定义的路径的话，需要在初始化的时候把路劲加入`Properties`里。例如：
<pre><code>
	Properties p = new Properties();
    p.setProperty("file.resource.loader.path", "/opt/templates");
    Velocity.init( p );
<code></pre>
