#使用`luaRocks`安装
``` 
luarocks install lua-resty-template
```
安装完成后会默认会被放到`/usr/local/share/lua/5.1/?.lua`下,而这个目录也是lua默认安装搜索的目录.所以,不用特别指定,直接使用
即可.
#结合`openresty`使用
在配置文件中添加如下:
``` 
http {
  server {
    # 指定模板的根路径,这样寻找模板就会在这个目录下寻找
    set $template_root /usr/local/openresty/nginx/html/templates;
    location / {
      root html;
      content_by_lua '
        local template = require "resty.template"
        template.render("view.html", { message = "Hello, World!" })
      ';      
    }
  }
}
```
编辑模板文件:`/usr/local/openresty/nginx/html/templates/view.html `
```
<!DOCTYPE html>
<html>
<body>
  <h1>{{message}}</h1>
</body>
</html>
```
最后返回的页面效果如下:
``` 
<!DOCTYPE html>
<html>
<body>
  <h1>Hello, World!</h1>
</body>
</html>
```