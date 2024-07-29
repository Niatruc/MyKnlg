* `nvm`
    * `root`: 打印nvm的安装目录
    * `ls`: 列出已安装的node
        * `ls available`
    * `install <版本号>`
        * 下载的node放在环境变量`NVM_HOME`指定的目录中. 
        * `install lts`
    * `use <版本号>`
        * 会以环境变量`NVM_SYMLINK`创建一个链接, 指向该版本node的目录. 
    * `arch 64`: 设置位数
    * `node_mirror <url>`: 设置node镜像, `<url>`为空则使用默认
        * `nvm node_mirror https://npmmirror.com/mirrors/node/`: 使用淘宝node镜像
    * `npm_mirror <url>`: 设置npm镜像, `<url>`为空则使用默认
        * `nvm npm_mirror https://npmmirror.com/mirrors/npm/`: 使用淘宝node镜像
    * ``: 

* `npm`
    * `install <pkg>`
        * `-g`: 全局安装
    * `config`
        * `get/set`
            * `userconfig`
            * `registry`: 镜像站点
            * `prefix`: 全局模块的存放路径
            * `cache`: cache的路径

# VUE
* 参考
    * [Vue3安装配置、开发环境搭建(组件安装卸载)（图文详细）](https://blog.csdn.net/weixin_69553582/article/details/129584587)
* 安装
    * `npm install vue -g`