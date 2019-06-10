###关于读取文件
>
- 配置文件以properties为例
- dao.js配置fields,fields里面的属性path指扫描的路径
- 要获取某一个配置文件里某个值`{java:"$conf.get('mail.HostName')"},`获取conf文件夹下的属性mail.HostName的值