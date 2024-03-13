# URL 匹配规则

Servlet 使用 `url-pattern` 标签指定一个 Servlet 的 URL 映射规则，容器通过 `url-pattern` 设定的规则匹配请求路径，从而找到对应的 Servlet

Servlet 规范定义了三种基本的匹配规则：

- 精确匹配
- 通配符匹配
- 扩展名匹配

## 匹配规则

### 精确匹配

`url-pattern` 中配置的项必须与 url 完全精确匹配

当配置信息如下时：

```xml
<servlet-mapping>
    <servlet-name>MyServlet</servlet-name>
    <url-pattern>/kata/detail.html</url-pattern>
    <url-pattern>/demo.html</url-pattern>
    <url-pattern>/table</url-pattern>
</servlet-mapping>
```

以下请求都会被匹配到：

```markdown
http://127.0.0.1/myapp/kata/detail.html
http://127.0.0.1/myapp/demo.html
http://127.0.0.1/myapp/table
```

<font color=red>`/` 是特殊的精确匹配符，它将匹配所有的 URL 路径</font>

### 扩展名匹配

匹配带有指定后缀的请求

当配置信息如下时：

```xml
<servlet-mapping>
    <servlet-name>MyServlet</servlet-name>
    <url-pattern>*.jsp</url-pattern>
</servlet-mapping>
```

以下请求会被匹配到：

```markdown
http://127.0.0.1/myapp/demo.jsp
http://127.0.0.1/myapp/test.jsp
```

### 路径匹配

匹配以指定路径开始的请求

当配置信息如下时：

```xml
<servlet-mapping>
    <servlet-name>MyServlet</servlet-name>
    <url-pattern>/kata/*</url-pattern>
</servlet-mapping>
```

则请求的 url 的路径是以 `/kata` 开始，就都会被匹配：

```markdown
http://127.0.0.1/myapp/kata/demo.html
http://127.0.0.1/myapp/kata/test.jsp
http://127.0.0.1/myapp/kata/test/detail.html
http://127.0.0.1/myapp/kata/action
```

<font color=red>`/*` 是特殊的精确匹配符，它将匹配所有的 URL 路径</font>

### 优先级

精确匹配 > 扩展名匹配 > 路径匹配

## 

