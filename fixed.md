# 问题解决记录



#### [mysql] text类型字段提示：doesn't have a default value

解决方法：修改/etc/my.cnf.d/server.cnf文件，在[mysqld]下面添加一行：

```bash
[mysqld]
sql_mode=NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```



#### [小程序] 无法解析html实体

小程序无法解析html实体，例如：&#44592;&#54924;&#51032; &#46263;&#47732;这种韩文编码，故封装js方法解决：

```javascript
function parseEntity(str) {
    let str_ary = str.match(/&#\d*?;/g);
    if (!str_ary) {
        return str;
    }
    for (s of str_ary) {
        let rs = String.fromCharCode(s.replace(/[&#;]/g, ''));
        str = str.replace(s, rs);
    }
    return str;
}

let str = '你好&#44592;&#54924;&#51032;&#46263;&#47732;';
console.log(parseEntity(str));
```

