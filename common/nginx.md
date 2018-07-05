MAC系统升级到 10.13.5  
 
重装homebrew后 发现nginx无法使用   

``` script
brew update && brew upgrade && brew rm extempore && brew install extempore
```

修复nginx后 启动原来的项目发现在webpack的dev server上访问的index.js http返回200缺是空内容   

于是在重新升级脚手架和对应的第三方库后发现依然如此,修改webpack配置后依然无效 排除webpack影响   

然后看nginx ERR_CONTENT_LENGTH_MISMATCH这个错误      
主要是由于nginx的 proxy_pass 的 proxy_temp文件夹权限所导致的？   
1. 找到对应文件夹/usr/local/var/run/nginx 一顿chown chmod无效   
2. 继续寻找 /usr/local/var/run/nginx/proxy_temp 由于是nobody又是一顿chown chmod无效
3. 修改nginx config    proxy_max_temp_file_size 0也无效   
开始怀疑人生   

最后觉得可能是proxy_temp文件夹下的历史文件导致的影响，清空了proxy_temp，恢复正常