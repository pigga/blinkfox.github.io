Nginx出现的`413 Request Entity Too Large`错误,这个错误一般在上传文件的时候出现，打开nginx主配置文件nginx.conf，找到http{}段，添加
**解决方法**就是<br/>

打开nginx主配置文件nginx.conf，一般在`/usr/local/nginx/conf/nginx.conf`这个位置，找到http{}段，修改或者添加
```
client_max_body_size 500m;
```
然后重启nginx，
```
nginx -s reload
```