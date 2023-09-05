* 参考
    * https://www.yiibai.com/tcl

* Tcl命令: `commandName argument1 argument2 ... argumentN`
* `expr <算术表达式>`
* 设置变量: `set v1 <数值>`
* 数据类型
    * 字符串
    * 列表: 
        * `"a b c"`或`{a b c}`
        * 下标访问: `[lindex $my_list <下标>]`
    * 关联数组: 
        ```t
        set m(a) 80
        puts $m(a) # 80
        ```
    * 句柄: 
        * `set myfile [open "filename" r]`