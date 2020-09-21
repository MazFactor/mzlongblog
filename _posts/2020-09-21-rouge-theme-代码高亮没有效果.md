---
layout: post
title: rouge-theme 代码高亮无效果的问题
categories: blog
description: 解决rouge theme代码高亮的问题
keywords: Rouge-theme 代码高亮
topmost: true
---
markdown中代码块以“\`\`\`”（锐音符）开始和结尾，但是为了告诉markdown接下来的代码到底是java还是C#，或者是html则需要在开始的“\`\`\`”（锐音符）后面加上代码类型，如“```java”。如若不加则没有代码高亮效果...<!-- more -->。如下所示：

```
package com.viewscenes.netsupervisor.spi;
public interface SPIService {
    void execute();
}
```

加上java后的效果则是这样：

```java
package com.viewscenes.netsupervisor.spi;
public interface SPIService {
    void execute();
}
```

刚开始使用jekyll + github pages + rouge的时候遇到的问题，后来看了别人写的文章后大悟，特此记下以备后用！