---

title: 常见的 Web Safe 攻防战 
date: 2018-08-12 16:30:00
tags: [Web, Security, XSS, CSRF]
excerpt: Web 安全演示分享

---

> 本次分享的目的，重点在于，让大家对常见的 Web 攻击有一个直观的认知。
>
> 当你知道，黑客是通过什么样的漏洞来攻击你的时候，你就更容易理解为什么要通过这样的方式来防范黑客攻击
>
> 本次分享不会过多涉及概念、防范的细节、千奇百怪的攻击方式。
>
> 有兴趣的同学，可以去找 谷G 同学请教。


## 演示地址


[用户端](http://web-safe-demo.herokuapp.com/login)  ([SourceCode](https://github.com/kenneth-hao/web-safe-demo))

[Hack 端](http://web-safe-demo-hack.herokuapp.com) ([SourceCode](https://github.com/kenneth-hao/web-safe-demo-hack))


## Tools


> 高版本 Chrome 会自动识别部分 XSS 攻击，并禁止你的访问
```sh
# 为了方便演示效果，请关闭目前已打开的 Chrome 浏览器，并使用下面的命令重新启动
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --args --disable-xss-auditor
```

<!-- more -->


## XSS


> 后端在入库前应该选择不相信任何前端数据，将所有的字段统一进行转义处理。
>
> 前端在渲染页面 DOM 的时候应该选择不相信任何后端数据，任何字段都需要做转义处理。

### 攻击原理

浏览器可以动态加载、渲染页面 DOM，并执行 JS 脚本。

### In Practice

> Demo - 公开课案例

那么在我们的项目中，是怎么来防范 XSS 攻击的呢？

因为我们已经完全实现了前后端分离，所以我们重点关注服务器端的实现。

在我们的 Web 应用中，没有直接渲染页面的部分，所以，输出过滤也没有；只针对 API 参数的输入做了 XSS 的过滤。

#### Show The Code

```java
public abstract class AskBaseController extends RootController {

  // 默认开启
  public boolean isAntiXssEnabled() {
    return true;
  }
  
  @InitBinder
  public void commonInitBinder(final WebDataBinder binder) {
    if (isAntiXssEnabled()) {
      binder.registerCustomEditor(String.class, new PropertyEditorSupport() {
        @Override
        public void setAsText(String text) {
          setValue(WebUtil.escapeHtml(text));
        }

        @Override
        public String getAsText() {
          return ObjectUtil.nullToDefault(getValue(), StringConsts.EMPTY).toString();
        }
      });
    }
  }
  
}


public class WebUtil {
  
  public static String escapeHtml(String source) {
    if (StringUtils.isNotEmpty(source)) {
      StringBuilder buff = new StringBuilder((int)((double)source.length() * 1.3D));

      for(int i = 0; i < source.length(); ++i) {
        char _char = source.charAt(i);
        switch(_char) {
          case '"':
            buff.append("&quot;");
            break;
          case '&':
            if(url) {
              buff.append(_char);
            } else {
              buff.append("&amp;");
            }
            break;
          case '\'':
            buff.append("&#39;");
            break;
          case '<':
            buff.append("&lt;");
            break;
          case '>':
            buff.append("&gt;");
            break;
          default:
            buff.append(_char);
      }
    }
    return buff.toString();
  }
  return source;
}

```

#### 缺陷

- 以 Controller 为单位，如果设置为关闭，该 Controller 下的所有接口默认都会取消 XSS 的输入过滤
- 依赖于 Spring MVC 容器实现，如果引入其他 MVC 框架，需要重新实现。


## CSRF

> 在服务器端接收到的请求后，去校验这个请求是可信任的

### 攻击原理

<iframe> 是浏览器早点支持的标准规范之一

iframe 内嵌的网页可以是第三方的完全独立的站点，与当前父容器站点关联很小。

在 iframe 标签内部的表单提交，默认会当做是第三方站点发出的请求，与当前父容器无关。

### In Practice

然后，我找了下我们项目中防范 CSRF 的方式。。

Em……….. 

成功攻击目标。。

怎么防范，就留个课后作业吧。。

## SQL 注入

> 只要不拼接 SQL，怎么样都行
>
> 脚本语言（PHP、Python），要特别注意！！




## REF

推荐阅读：[常见的 Web 安全攻防总结](https://zoumiaojiang.com/article/common-web-security/)
