#### 1.DispatchServlet的doDispatch方法

```java
this.processDispatchResult(processedRequest, response, mappedHandler, err, (Exception)dispatchException);
```

#### 2.DispatchServlet的processDispatchResult方法

```java
this.render(mv, request, response);
```

#### 3.DispatchServlet的render方法

```java
//FreeMarkerView
view = this.resolveViewName(mv.getViewName(), mv.getModelInternal(), locale, request);
view.render(mv.getModelInternal(), request, response);
```

#### 4.AbstractView的render方法

```java
this.renderMergedOutputModel(mergedModel, this.getRequestToExpose(request), response);
```

#### 5.AbstractTemplateView的renderMergedOutputModel方法

```java
//exposeRequestAttributes为true进入判断
if(this.exposeRequestAttributes) {
    for(Enumeration session = request.getAttributeNames(); session.hasMoreElements(); model.put(en, attribute)) {
        en = (String)session.nextElement();
        //如果model和request中相同的属性名并且allowRequestOverride为false就抛出异常
        if(model.containsKey(en) && !this.allowRequestOverride) {
            throw new ServletException("Cannot expose request attribute \'" + en + "\' because of an existing model object of the same name");
        }
        attribute = request.getAttribute(en);
    }
}
```

---