---
title: JDK8 lambda reduce （转载自zooooooooy）
---

今天接了一个需求，需要在302的同时，将代理的所有参数都在转发到新的页面上

直接上代码

```
	Map<String, String[]> parameterMap = request.getParameterMap();
    List<String> paramList = new ArrayList<>();
    parameterMap.forEach((k,v) -> {
       paramList.add(k + "=" + v[0]);
    });

    String paramChain = paramList.stream().reduce((s1, s2) -> s1 + "&" + s2).get();
    response.setStatus(302);
    response.setHeader("Location", lxAddress + "?" + paramChain);
    response.setHeader("Connection", "close");
```

