# 配置开发环境
## vscode
* 配置定义跳转
    1. 打开插件扩展, 安装`Ruby Peng Lv`
    2. 打开设置(`Preferences: Settings`)
    3. 添加配置: 
        ```json
        "ruby.useLanguageServer": true, 
        "ruby.intellisense": "rubyLocate", 
        ```
* jupyter
    * 安装libzmq
        * 到github下载源码
        * `mkdir build && cd build && cmake ..`
        * `make && sudo make install`
    * `gem install iruby`
    * `iruby register --force`
## rbenv
* 安装的环境默认在`~/.rbenv/versions`下. 

## 包管理
* bundle
    * Gemfile

## 全局变量和方法
* `$LOAD_PATH`: `require`和`load`的搜索路径
* `$LOADED_FEATURES`: 保存已经require过的文件
* `__FILE__`: 当前文件
* `__LINE__`: 当前行

## 元编程
* 所有类的所属类都是`Class`, 其也成为元类. 
* `include`和`prepend`
    * prepend将模块插入祖先链中当前类的下方(**ancestors列表中靠前的地方**), include则插入上方. 
    * 类会先从prepend插入的模块中寻找方法, 再找当前类定义的方法, 再到include的模块中找. 
* 打开`Kernel`模块并添加的方法称为内核方法. 这些方法是所有对象共用的. 
* Ruby程序启动时, 会创建顶层对象`main`. 
    * 顶级变量: 顶层对象`main`的实例变量. 直接在最外层用`@var1`表示. 
* `refine`: 只作用于希望生效的地方
```rb
module M
    refine String do
        def f1; "" end
    end
end

using M # 将M引入当前作用域
```

* 动态方法: 
    * 动态调用方法: `myObj.send(:method1, arg1)`
    * 动态定义方法: 
    ```rb
    class C
        define_method :method1 do |arg1|
            ...
        end
    end
    ```
    * 动态代理方法(幽灵方法): 
    ```rb
    class C
        def method_missing(method, *args) # 实例找不到方法是
        ...
        end
    end
    ```
* 查询方法是否可用: `myObj.respond_to? :method1`
    * 当询问的是幽灵方法时, `respond_to?`会调用`respond_to_missing(method, include_private = false)`
* 白板类`BasicObject`: 
    * 继承该类后, `p`, `throw`, `raise`都用不了. 
* 作用域
    * 作用域门: `class`, `module`, `def`等关键字可打开作用域门. 
        * `class`关键字意味着进入类的上下文中. 
    * 闭包/代码块
        * 创建代码块时也会创建绑定
        * 闭包还会捕获当前环境中的绑定
    * 扁平作用域: 让外部变量穿越作用域门, 被另一个作用域访问. 
        * `class`用`C = Class.new {...}`代替. 
        * `def`用`define_method :method1 {...} `代替. 
        * 上下文探针
            ```rb
            myObj.instance_eval do
                self # 指向代码块的接收者, 即myObj
                @v # myObj的实例变量
                # 还可访问私有方法
                # 可访问代码块外的变量, 且外部变量会覆盖实例的同名访问器
            end
            ```

### 元方法
* `__method__`: 当前所在方法名
* `__callee__`: 当前方法的方法名
* `binding`: 产生一个指向当前闭包的`Binding`对象. 
* `BasicObject`
    * `#instance_eval {...}`: 进入实例上下文并运行代码
* `Object`
    * `#class`: 实例所属类
    * `#instance_methods`: 所有实例方法
    * `#instance_variables`: 所有实例变量
* `Class`
    * `#superclass`: 父类
* `Module`
    * `#ancestors`: 模块的祖先链. 列表最后面的为继承的终点. 
    * `#constants`: 模块中包含的常量
    * `#undef_method`: 删除方法
    * `#remove_method`: 删除方法
* `Kernel`
    * `.block_given?`: 询问当前方法调用中是否含有块. 
    * `.local_variables`: 所有局部变量. 