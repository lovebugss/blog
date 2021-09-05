---
title: 使用Postman 进行新老接口数据diff
tags: 
    - postman
    - diff
categories: TEST
---
![Blog_Postman2_1200x600](https://i.loli.net/2021/09/05/cyHrofv5zgKTaQx.png)


> 背景: 公司最近需要将原有php服务使用java进行重构. 为了保证重构后的接口正确性, 需要对数据进行对比, 保证数据的完全一致. 所以就需要对数据进行对比.
> 大致思路是: 再发送请求之前, 通过修改host, 先获取旧得数据, 然后放入环境变量. 然后再使用jsondiffpatch工具进行数据对比.
<!--more -->
## 在Postman中引入jsondiffpatch

Postman 支持js语法, 本身就有封装node环境. 所以可以引入第三方js库.

具体方法如下:

1. 在请求之前, 获取jsondiffpatch代码, 然后放入环境变量.

    ![Untitled](https://i.loli.net/2021/09/05/tyNPwzJKVi7S4Hl.png)

    ```jsx
    // 当运行多个测试用例时, 会导致文件拉去多次. 
    // 这里做个判断, 先检查环境变量中有没有, 若有就不重新拉取
    var code = pm.environment.get("jsondiffpatch_code")
    if(!code){
        // 引入jsondiffpatch.js
        pm.sendRequest("https://cdn.jsdelivr.net/npm/jsondiffpatch/dist/jsondiffpatch.umd.min.js", (err, res)=>{
            if(err){
                console.log("error: " + err);
            }else{
                pm.environment.set("jsondiffpatch_code", res.text());
            }
        })
    }
    ```

2. 执行jsondiffpatch代码
![Untitled 1](https://i.loli.net/2021/09/05/HanOXY7QvRst94F.png)


    ```jsx
    // 引入
    let code = pm.environment.get("jsondiffpatch_code");
    if(code){
       (new Function(code))() 
    }
    ```

    引入之后, 就可以使用jsondiffpatch了.

## 环境变量

```jsx
*host: 新服务host
*old_host: 老服务host
jsondiffpatch_code: 用来存储jsondiffpatch代码.
old_data: 存放就接口数据.
```

## 完整代码

Pre-request-script

```jsx
// 当运行多个测试用例时, 会导致文件拉去多次. 
// 这里做个判断, 先检查环境变量中有没有, 若有就不重新拉去
var code = pm.environment.get("jsondiffpatch_code")
if(!code){
    // 引入jsondiffpatch.js
    pm.sendRequest("https://cdn.jsdelivr.net/npm/jsondiffpatch/dist/jsondiffpatch.umd.min.js", (err, res)=>{
        // (new Function(res.text()))()
        pm.environment.set("jsondiffpatch_code", res.text());
        
    })
}

console.log("当前url: " + pm.request.url)
// 1. 拉去旧数据, 然后进行缓存.
// 将原来的host 缓存
let host = pm.environment.get("host");
// 获取host, 为旧服务器
let oldHost = pm.environment.get("old_host");
let url = pm.request.url;
let oldUrl = url.toString().replace("{{host}}", oldHost)
let method = pm.request.method;

// 发送请求
let requestParam = {
    url: oldUrl,
    method: pm.request.method,
    header: {...pm.request.headers},
    body: {...pm.request.body}
}

pm.sendRequest(requestParam, (error, response) => {
  if(error){
    console.log(`request url: [${oldUrl}], error: ${error}`)
  }else{
      // 放入环境变量
     pm.environment.set("old_data", response.json()) 
  }
});
```

Test:

```jsx
// 引入
(new Function(pm.environment.get("jsondiffpatch_code")))()

// 2. 对比结果集是否一致
// 旧数据
let oldData = pm.environment.get("old_data")

// 如果没有拉取到旧数据, 就不进行对比. 不影响后面的逻辑
if(oldData){
    // 新数据
    let data = pm.response.json();
    // 数据对比
    var delta = jsondiffpatch.diff(oldData, data);
    pm.test("data diff", ()=>{
    if(delta){
        console.log(`数据不一致, 差异数据: ${JSON.stringify(delta)}`)
        pm.expect("数据不一致").throw();
    }
    })
}
// 删除旧数据
pm.environment.unset("old_data")
```

注意: 只需要在最外层引入即可.
![Untitled 2](https://i.loli.net/2021/09/05/W3dCbwNLnuBIPmf.png)


参考:

[https://github.com/benjamine/jsondiffpatch](https://github.com/benjamine/jsondiffpatch)

[https://learning.postman.com/docs/writing-scripts/script-references/postman-sandbox-api-reference/](https://learning.postman.com/docs/writing-scripts/script-references/postman-sandbox-api-reference/)

[https://community.postman.com/t/adding-external-libraries-to-postman/1971/20](https://community.postman.com/t/adding-external-libraries-to-postman/1971/20)