---
title: SpringMVC 系列| 视图解析器 InternalResourceViewResolver
date: 2022-04-28 17:08:31
tags: 
    - SpringMVC
    - 后端
categories: SpringMVC
description: 视图解析器通过内部服务器进行转发的形式访问资源，解决了由于安全问题考虑将JSP放置在/WEB-INF/目录下使得外部不能直接访问的情况

---
<meta name="referrer" content="no-referrer"/>

![题图](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/img/2022/03/202203021639302.jpg)
<!--more-->

## 配置文件 springmvc.xml

​	为了使用 InternalResourceViewResolver 我们都会在 SpringMVC 的配置文件中进行如下配置

```xml
<!--  自定义视图解析器  -->
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

然后我们想访问一个WEB-INF目录下的文件就可以直接输入这个文件的名字即可

例如：view  视图解析器就会在底层帮我们解析为 /WEB-INF/view.jsp 

那么，它的底层究竟是如何来实现的呢？

------
 
## 底层实现
InternalResourceViewResolver：它是UrlBasedViewResolver的子类，那么也就是说UrlBasedViewResolver所有的特性它全部支持，

　　那么InternalResourceViewResolver到底有什么特性呢？我们从它的字面意义上来看，可以理解为内部资源视图解析器，也正是如此，它也是应用最广泛的视图解析器。

![img](https://img2018.cnblogs.com/blog/1760573/201909/1760573-20190913171016769-311318391.png)

 

 

 

来让我们来看一下，它的底层源码



```
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.web.servlet.view;

import org.springframework.util.ClassUtils;

public class InternalResourceViewResolver extends UrlBasedViewResolver {
    private static final boolean jstlPresent = ClassUtils.isPresent("javax.servlet.jsp.jstl.core.Config", InternalResourceViewResolver.class.getClassLoader());
    private Boolean alwaysInclude;

    public InternalResourceViewResolver() {
        Class<?> viewClass = this.requiredViewClass();
        if (viewClass.equals(InternalResourceView.class) && jstlPresent) {
            viewClass = JstlView.class;
        }

        this.setViewClass(viewClass);
    }

    protected Class<?> requiredViewClass() {
        return InternalResourceView.class;
    }

    public void setAlwaysInclude(boolean alwaysInclude) {
        this.alwaysInclude = alwaysInclude;
    }

    protected AbstractUrlBasedView buildView(String viewName) throws Exception {
        InternalResourceView view = (InternalResourceView)super.buildView(viewName);
        if (this.alwaysInclude != null) {
            view.setAlwaysInclude(this.alwaysInclude);
        }

        view.setPreventDispatchLoop(true);
        return view;
    }
}
```



InternalResourceViewResolver会通过 执行buildView方法然后调用父类的vuildView方法，把我们返回的请求或返回的viewname传过去，我们来看下这个方法的实现

```
//将不重要的代码部分已删除 protected AbstractUrlBasedView buildView(String viewName) throws Exception {
        AbstractUrlBasedView view = (AbstractUrlBasedView)BeanUtils.instantiateClass(this.getViewClass());//获得一个视图类  有继承关系
        view.setUrl(this.getPrefix() + viewName + this.getSuffix());//获取我们在配置文件中配置的prefix 和suffix和传进来的viewName 
        String contentType = this.getContentType();
        if (contentType != null) {
            view.setContentType(contentType);//视图类型
        }

        return view;//返回我们的视图
    }
```

我们通过这个方法可以发现，首选这个方法创建了一个视图，虽然我们不认识，但是他们间接的有继承关系，我们可以自行查看继承结构。

然后就是获取我们在SpringMVC中配置的InternalResourceViewResolver的prefix和suffix还有viewName名，构成一个完整的url例如：/WEB-INF/a.jsp，最后把构成的视图返回构成了一个

```
InternalResourceView视图。然后InternalResourceView视图会把Controller处理器返回的模型属性全部都放到HttpServletRequest里面，让我们看下底层的执行
```



```
//调用的是InternalResourceView对象的方法

 protected void renderMergedOutputModel(Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
        this.exposeModelAsRequestAttributes(model, request);//通过调用这个方法然后执行放置到request，看接下来下面片段的代码
        this.exposeHelpers(request);
        String dispatcherPath = this.prepareForRendering(request, response);
        RequestDispatcher rd = this.getRequestDispatcher(request, dispatcherPath);
        if (rd == null) {
            throw new ServletException("Could not get RequestDispatcher for [" + this.getUrl() + "]: Check that the corresponding file exists within your web application archive!");
        } else {
            if (this.useInclude(request, response)) {
                response.setContentType(this.getContentType());
                if (this.logger.isDebugEnabled()) {
                    this.logger.debug("Including resource [" + this.getUrl() + "] in InternalResourceView '" + this.getBeanName() + "'");
                }

                rd.include(request, response);
            } else {
                if (this.logger.isDebugEnabled()) {
                    this.logger.debug("Forwarding to resource [" + this.getUrl() + "] in InternalResourceView '" + this.getBeanName() + "'");
                }

                rd.forward(request, response);
            }

        }
    }

//调用的是AbstractView 类

protected void exposeModelAsRequestAttributes(Map<String, Object> model, HttpServletRequest request) throws Exception {
        Iterator var3 = model.entrySet().iterator();

        while(var3.hasNext()) {
            Entry<String, Object> entry = (Entry)var3.next();
            String modelName = (String)entry.getKey();
            Object modelValue = entry.getValue();
            if (modelValue != null) {
                request.setAttribute(modelName, modelValue);//把Controller返回的模型属性值放入
                if (this.logger.isDebugEnabled()) {
                    this.logger.debug("Added model object '" + modelName + "' of type [" + modelValue.getClass().getName() + "] to request in view with name '" + this.getBeanName() + "'");
                }
            } else {
                request.removeAttribute(modelName);
                if (this.logger.isDebugEnabled()) {
                    this.logger.debug("Removed model object '" + modelName + "' from request in view with name '" + this.getBeanName() + "'");
                }
            }
        }

    }
```

[![复制代码](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/img/2022/03/202204281715735.gif)](javascript:void(0);)

**然后通过RequestDispatcher在服务器端把请求forword重定向到目标URL**

**以上就是InternalResourceViewResolver解析流程**

**连贯起来就是**

InternalResourceViewResolver会把返回的视图名称都解析为InternalResourceView对象，InternalResourceView会把Controller处理器方法返回的模型属性都存放到对应的request属性中，然后通过RequestDispatcher在服务器端把请求forword重定向到目标URL。比如在InternalResourceViewResolver中定义了prefix=/WEB-INF/，suffix=.jsp，然后请求的Controller处理器方法返回的视图名称为test，那么这个时候InternalResourceViewResolver就会把test解析为一个InternalResourceView对象，先把返回的模型属性都存放到对应的HttpServletRequest属性中，然后利用RequestDispatcher在服务器端把请求forword到/WEB-INF/a.jsp。

###  

### **最后我们在总结下总体的视图解析流程：**

1、调用目标方法，SpringMVC将目标方法返回的String、View、ModelMap或是ModelAndView都转换为一个ModelAndView对象；

2、然后通过视图解析器（ViewResolver）对ModelAndView对象中的View对象进行解析，将该逻辑视图View对象解析为一个物理视图View对象；

3、最后调用物理视图View对象的render()方法进行视图渲染，得到响应结果。