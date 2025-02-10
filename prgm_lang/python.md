# 环境
* 下载: `http://python.org/ftp/python/`
* venv
    * 只是一个内置于python3的模块. 不能指定虚拟环境的python版本. 
    * `python3 -m venv /path/to/new/vitual/env`
* virtualenv
    * pip安装
* vscode
    * 配置notebook
        * 在`setting.json`中添加配置: 
            ```json
                "python.venvPath": "<包含venv环境的目录>", 
                "sudo": true
            ```
    * 配置使用conda环境
        * 打开全局配置文件`settings.json`, 新增配置项: `"python.condaPath: "<conda程序的绝对路径>"`; 重启vscode. 
        * 解决无法使用python2进行调试的问题(`An Invalid Python Interpreter Is Selected`)
            * 在扩展中找到`Python`, 点击`Uninstall`键右侧的下拉键, 点击`Install Another Version`, 选择`v2021.9.1246542782`(这个插件对Python2的支持到这个版本结束). 
        * `launch.json`: 
            * `args`: 值为列表, 列表中每一项对应一个命令行参数. 
            * `justMyCode`: 将该值设为`false`, 这样调试的时候调用栈上方能显示第三方库的函数. 
* jupyter(lab)
    * 安装kernel: 在目标python环境中执行: 
        * `pip install ipykernel`
        * `python -m ipykernel install --user --name=<kernel名>`
    * 卸载kernel: `jupyter kernelspec remove <kernel名>`
    * 设置自动重载模块: 
        > `%load_ext autoload`
        > `%autoload 2`
    * 快捷键
        * `shift + tab`: 查看函数描述(参数, 所在文件等)
    * 问题
        * `IOPub data rate exceeded`
            * `jupyter --config-dir`: 找到配置文件目录
            * 编辑`jupyter_notebook_config.py`(没有则新建): 
                * 添加: `c.NotebookApp.iopub_data_rate_limit = 10000000`(适当加大此值)
* pip
    * `install <模块> -i https://pypi.tuna.tsinghua.edu.cn/simple`
* conda
    * 命令
        * `conda`
            * `config`: 首次使用后, 会在用户目录下生成`.condarc`文件. 
                * `--add`
                    * `envs_dirs <路径>`: 添加环境路径. 以后创建和搜索虚拟环境时, 会现在该路径下执行操作. 
            * `env`: 
                * `list`: 列出所有环境
            * `create`: 
                * `--name <虚拟环境名称>`: 
                * `--prefix <虚拟环境完整路径>`: 如果不想装在默认的`.conda`目录, 则不要用`--name`选项, 而是用这个. 
                * `python=<版本号>`
            * `remove -n <虚拟环境名称>`: 
            * 离线创建环境
                * 执行命令: `conda create -n [name] --clone [filepath] --offline`
                    * 出错: `This command is using a remote connection in offline mode.`
    * 打包和迁移
        * `pip install conda-pack`
        * `conda pack -n my_env -o my_env.tar.gz`: 打包
        * 在另一台机子上解压`my_env.tar.gz`, 将目录拷贝到`miniconda3/envs`目录下. 

    * mamba
        * 安装: `conda install -c conda-forge`
        * 创建空白环境: `mamba create -n <环境名>`

# 打包
* 参考
    * [如何一键保护你的 Python 代码？](https://blog.csdn.net/dsuiofh/article/details/137555769)
* pyinstaller
    * 要点
        * 会先在当前目录(或`--specpath`指定的目录)生成一个`.spec`文件, 一个`build`目录和一个`dist`目录(存放最终的可执行文件). 
        * 可用`pyi-makespec`仅生成`.spec`文件. 
    * 基本用法
        ```sh
            # 参考: https://pyinstaller.org/en/stable/usage.html

            pyinstaller
                -F # 表示生成单个可执行文件
                -D # 表示打包成目录. (会生成一个main程序和一个_internal目录, 后者存放依赖文件)
                -n <名称> # 指定spec文件和打包程序的名称
                -p <目录> # 指定import时查找的目录(类似于PYTHONPATH)
                --workpath <目录> # 指定临时文件存放目录(默认是`./build`)
                --distpath <目录> # 指定打包程序存放目录(默认是`./dist`)
                --hidden-import <模块名> # hiddenimport是指那些不是通过import导入模块的操作(比如)
                --runtime-tmpdir <目录> # 配合`-F`使用, 指定运行时解压依赖文件的临时路径. 若未指定, 则为`/tmp/_MEI*`
                -s # 去除符号表
                main.py # 目标文件

            # 也可以直接使用生成的spec文件: 
            pyinstaller main.spec
        ```
    * 问题
        * 打包后的程序的运行问题: 
            * `ModuleNotFoundError: No module named xxx`
                * 分析: 无法导入工程中其他模块
                * 解决方法: 加选项`--path .`, 将当前目录(即工程目录)指定为import的查找目录. 
            * `fail to load the dynamic library`
                * 原因: 有些模块会加载动态库
                * 解决方法(以`unicorn`等为例): 
                    * 方法一: 加选项`--collect-all unicorn`
                    * 方法二: 可以修改spec文件: 
                        ```py
                            from PyInstaller.utils.hooks import collect_dynamic_libs # 导入这个函数

                            a = Analysis(
                                ...,
                                binaries= # 效果等同于`--collect-binaries`
                                    collect_dynamic_libs("unicorn") + \
                                    collect_dynamic_libs("unicornafl") + \
                                    collect_dynamic_libs("capstone") + \
                                    collect_dynamic_libs("keystone"),
                                hiddenimports=["unicorn", "unicornafl", "capstone", "keystone"],
                                ...,
                            )

                        ```
            * `configparser.NoSectionError: No section:`
                * 分析
                    * 程序导入了qiling模块
                    * 初始化时, 使用了`utils.py:profile_setup`函数, 其中有一行`qiling_home = Path(inspect.getfile(inspect.currentframe())).parent`
                    * 调试发现打包程序运行`inspect.getfile(inspect.currentframe())`返回的是`qiling/utils`(在没打包时, 获取的是绝对路径); `qiling_home`值为`qiling`
                    * `ConfigParser`实例使用上面计算得到的路径去读取`qiling/profiles/linux.ql`文件, 找不到. 
                * 解决方法
                    * 方法一: 
                        1. 改用`-D`选项, 打包生成目录. 
                        2. 在打包后的程序运行前, 在它的工作目录下创建软链接指向`qiling`目录. 
                    * 方法二:
                        * 修改python程序, 使用`sys._MEIPASS`获取解压生成的临时目录, 然后同上所述建立软链接. 
# 文档
## sphinx
* 参考
    * [Build project documentation quickly with the Sphinx Python](https://medium.com/@pratikdomadiya123/build-project-documentation-quickly-with-the-sphinx-python-2a9732b66594)
    * [[野火]sphinx文档规范与模版](https://ebf-contribute-guide.readthedocs.io/zh-cn/latest/index.html)

* 基本步骤
    * 安装: `pip install sphinx sphinx_rtd_theme`
    * 创建一个要存放文档的目录(后称`docs`), 然后切换到该目录. 
    * 运行`sphinx-quickstart`, 在对话中填写项目信息. 之后会在docs下生成: 
        * `_build/`
            * `doctrees/`
            * `html/`
                * `static/`: 包含前端文件
                * `index.html`: 文档的入口. 
        * `_static/`
        * `_templates/`
        * `conf.py`: 记录Sphinx项目的配置. 
        * `index.rst`: Sphinx项目的根文档. 包含`toctree`(内容表树)的根. 
        * `Makefile`: 用于make
    * 修改`conf.py`:
        ```py
            # 添加: 
            import os
            import sys
            sys.path.insert(0, os.path.abspath('<项目目录>'))

            # 修改: 
            extensions = [
                'sphinx.ext.autodoc',
                'sphinx.ext.viewcode',
                'sphinx.ext.napoleon'
            ]

            # 添加: 
            html_theme = 'sphinx_rtd_theme'

            # 添加
            autodoc_default_options = {
                'members': True,
                'special-members': '__init__',
            }
        ```
    * `sphinx-apidoc -o docs <项目目录> <排除文件(可用通配符)>`: 会给每个模块生成`<模块名>.rst`; 另有一个`module.rst`
        * 注: 对于目录, 要确保有`__init__.py`文件(表示其为一个`package`), 否则`sphinx-apidoc`不会为其生成rst文件. 
    * 修改`index.rst`, 在其中引用`modules.rst`:
        ```rst
            .. toctree::
                :maxdepth: 2
                :caption: Contents:

                modules
        ```
    * 在docs目录下执行`make html`, 生成前端页面. 
* 支持markdown
    * `pip install -U myst-parser`
    * 在`conf.py`的`extensions`列表中添加`'myst_parser'`. 
    * 可将指定后缀的文件识别为markdown文件. 比如, 要将`.txt`文件识别为markdown文件, 可修改`conf.py`: 
        ```py
            source_suffix = {
                '.rst': 'restructuredtext',
                '.txt': 'markdown', # 添加此行
                '.md': 'markdown',
            }
        ```
    * 在`rst`文件中:
        * 可在`toctree`节点下, 添加要渲染的md文件路径. 
        * 可在其它节下面添加引入md文件的节: 
            ```sh
                .. include:: ../tutorial.md
                    :parser: myst # 或者commonmark
            ```
    * 交叉引用: 参考`https://myst-parser.readthedocs.io/en/latest/syntax/cross-referencing.html#heading-target`
        * 设置锚点: 
            ```
                (heading-target)=
                ### Heading
            ```
        * 引用: `[引用](#heading-target)`
    * 在markdown文件新增语法: 
        * 在md中渲染rst: 
            ```
                ```{eval-rst}
                    可以这样引用sphinx生成的方法: :func:`my_mod.func1`
                ```
            ```
* latexpdf
    * 安装相关包: `sudo apt install texlive-xetex texlive-lang-chinese latexmk`
* rst: `reStructuredText`标记语言
    ```sh
        .. automodule:: my_module
            :members:
            :no-undoc-members: # 排除没有docstring的类和函数等. 反之为 :undoc-members:
            :exclude-members: PrivateClass, private_function, *private_* # 指定排除的模块或行数(不会生产文档)

        # 添加代码示例
        .. code-block:: python

            from my_module import MyClass

            my_class = MyClass()
            my_class.public_method()
    ```
* python docstrings
    ```py
        def f(a: dict):
            """f函数. 

            Args:
                a (dict): 参数a. 

            Examples:
                * 你好
                .. code-block:: python

                    def f2():
                        f()

                .. code-block:: json

                    {
                        "a": 1
                    }
            """
            pass
    ```
    * 可以使用markdown, 但是代码块不能用md的语法, 需用restructuredText, 比如`.. code-block:: python`, 且**要紧接着一个空行**, 且**后面的代码块需要多一个缩进**. 
# 基本数据类型
## 数值
```py
    int("0x123", 16) # 字符串转整数. 二参表示进制
```
## 字符串
```py
    r"D:\dir" # 反斜杠不转义

    # 格式化字符串
    f"his name is {name:20s}" # 变量`name`后的冒号后面是格式字符串

    # 多行字符串
    s = """
        def f():
            print("123")
    """

    import textwrap
    s = textwrap.dedent(s) # 去掉多余缩进(去除的缩进数量按缩进最少的那行算, 比如这里就是`def f():`行)

    # 用`repr`可以将对象转换成字符串
    print(repr("hello")) # => 'hello', 会带上引号

    # 将命令行参数字符串转成参数列表
    import shlex
    command_line = 'ls -l "file with spaces.txt"  ./some/directory'
    arguments = shlex.split(command_line)

    info = 'abca'
    info.find("a") # 搜索子串第一次出现的位置, 未找到则返回-1
    info.index("a") # 同上, 但未找到时抛出异常
```
* `print("hello", end="")`: 不换行打印

## 字节数据
```py
    bytes_len = 10
    bytes(bytes_len) # 生成一个长度为bytes_len的全零字节串

    # bytes类型字节串不能修改, 要能修改得用bytearray

    # 字节和字符串互转
    str(b'hello', 'utf-8') # 字节串转字符串
    b'hello'.decode('utf-8') # 字节串转字符串
    bytes('hello', 'utf-8') # 字符串转字节串
    'hello'.encode('utf-8') # 字符串转字节串

    # 将字节串转成16进制格式的字符串
    # 方法一: 
    import binascii
    binascii.hexlify(b"asdf").decode('ascii') # '61736400'
    # 方法二: 
    ' '.join(['{:02x}'.format(x) for x in b"asdf"]) # '61 73 64 66'
    # 方法三: 
    b"asdf".hex(" ")

    # 将16进制格式字符串转成字节串
    hex_text = "61 73 64 66"
    bytes.fromhex(hex_text)

    # 整数和字节数据互转
    integer_value = 1024
    big_endian_bytes = integer_value.to_bytes(2, 'big')  #=> b'\x04\x00'
    iv = int.from_bytes(big_endian_bytes, 'big')
```

## 列表
```py
    a1 = [1, 2, 3]
    a2 = [4, 5, 6]

    a1 = ["%02x" % x for x in a1] # 列表推导式
    list(map(lambda x: "%02x" % x, a1)) # 对列表的每个元素应用函数, 生成新列表

    del a[0] # 删除第0个元素

    # 迭代器
    it = iter([1, 3, 5, 7])
    for _ in range(8):
        res = next(it) # 通过迭代器获取下一个
        print(res)

    # 利用迭代器, 获取列表中第一个符合条件的值
    next((i for i in [1,2,3] if i > 2), -1)

    # (filter函数的一参为None时)去除列表中所有非True值, 包括空字符串, 0等
    list(filter(None, ["", " ", 123, 0])) # => [" ", 123]
    
    a1.extend(a2) # 合并列表

    list(set(a1).intersection(set(a2))) # 交集
    list(set(a1).union(set(a2))) # 并集
    list(set(a1).difference(set(a2))) # 差集

```
* `yield`: 生成器
    ```py
        def f():
            a = 1
            while 1:
                res = yield a
                a += 1
        
        g = f() # g是一个生成器. 
        next(g) # 调用生成器. 返回值为yield后的值
    ```

    * `f()`不会实际运行`f`的函数体
    * 调用`next`时`f`才开始执行. 遇到`yield`后, 返回`a`, 停止. 再次`next`才往下执行
    
    ```py
        import sys
        
        def fibonacci(n): # 生成器函数 - 斐波那契
            a, b, counter = 0, 1, 0
            while True:
                if (counter > n): 
                    return
                yield a # 函数每次运行到这里时, 返回一个迭代器, 并将a作为函数的返回值
                a, b = b, a + b
                counter += 1
        f = fibonacci(10) # f 是一个迭代器, 由生成器返回生成
        
        while True:
            try:
                print (next(f), end=" ")
            except StopIteration:
                sys.exit()
    ```
* `zip`函数: 
    ```py
        a = [1, 2, 3]
        b = [4, 5, 6]
        zipped = zip(a, b)
        list(zipped) # [(1, 4), (2, 5), (3, 6)]

        # 使用迭代器
        x = iter([1, 2, 3, 4, 5, 6, 7, 8, 9])
        print(list(zip(x, x, x))) # [(1, 2, 3), (4, 5, 6), (7, 8, 9)]
        print(list(zip(*[x] * 3))) # 同上
    ```
## 元组
## 字典
```py
    d = {'a': 1, 'b': 2}

    list(d.items()) # [('a', 1), ('b', 2)]

    del d['a'] # 删除键'a'
    
    # 与json互转
    import json
    j = json.dumps(d)
    d1 = json.loads(j)

    # 从json文件中读取数据到字典
    with open("my_data.json", "r") as f:
        my_dict = json.load(f)

    # 保存字典到json文件
    with open("my_data.json", "w") as f:
        json.dump(my_dict, f, indent=4)

    # 遍历
    for (key, value) in d.items():
        print ('key: ', key, 'value: ', value)
```

# 函数
# 模块
```py
    import my_module

    my_module.__file__ # 获取模块路径
```
# 异常处理
```py
    import traceback
    try:
        ...
    except Exception as e:
        traceback.print_exc()
        print(traceback.format_exc()) # 与上同
        tb = traceback.TracebackException.from_exception(e, capture_locals=True) # 打印参数值
    else: 
        pass
```
# 类型注解
```py
    from typing import *
    def test(a: str, b: int = 2) -> None:
        pass
    
    l: list[int] = []  # list[整型]
    t: tuple[int, str] = (1, '2')  # 元组的第一个元素是int类型, 第二个元素是str类型
    d: dict[str, int] = {'1': 1}

    type MyType = list[int] # 自定义类型注解, 有助于编程时的代码提示
```
# 面向对象
* 
    ```py
        class Student:
            __slots__ = ('name', 'age') # 限制对象属性
    ```
* `super`: 
    * 是用来解决多重继承问题的
    * `super(type[, object-or-type])`: `type`是父类, `object-or-type`一般是`self`
    * 在子类的函数中, 调用父类方法(如, `f1`), 需要这么写: `super().f1()`


# 元编程
* 内省
    * 获取一个对象的所有属性(包括方法): `dir(obj)`
    ```py
        import inspect

        # 获取当前行详细信息
        def debug_print(*args):
            current_frame = inspect.currentframe()
            caller_frame = current_frame.f_back
            file_name = caller_frame.f_code.co_filename
            line_number = caller_frame.f_lineno
            function_name = caller_frame.f_code.co_name
            current_time = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
            base_info = f"[{current_time}] {file_name}:{line_number} {function_name}: {" ".join(args)}"
            print(base_info)
    ```
* 执行动态代码: 
    * `eval(expression, globals=None, locals=None)`: `globals`和`locals`指定了闭包的作用域
* 函数作为对象: 
    ```py
        import inspect

        def f(a, b):
            function_object = globals()["f"] # 获取函数对象
            f.attr1 = "123"
            current_function = inspect.currentframe().f_code.co_name # 获取函数名

        f.__name__ # "f"
        f.__code__ # 函数的code对象

        from inspect import signature, Parameter
        param_list = list(signature(f).parameters.values()) # 获取参数列表

        param_list[0].name # 'a'

        # 修改函数的参数列表
        param_list.insert(0, inspect.Parameter("new_a1", inspect.Parameter.POSITIONAL_ONLY))
        new_signature = inspect.Signature(parameters=param_list)
        f.__signature__ = new_signature

        # 注:
        # def func(positional_args, keyword_args, *tuple_grp_args, **dict_kw_args):
        # 如上所示, 一个函数的参数类型分别是: 
        #   positional_args: 位置参数(POSITIONAL_ONLY)
        #   keyword_args: 关键字参数(KEYWORD_ONLY)
        #   tuple_grp_args: 可变位置参数(VAR_POSITIONAL)
        #   dict_kw_args: 可变关键字参数(VAR_KEYWORD)

        # 实现函数内"静态变量"
        def counter():
            if not hasattr(counter, 'count'):
                counter.count = 0
            counter.count += 1
            return counter.count

        print(counter())  # Output: 1
        print(counter())  # Output: 2
    ```
* `mixin`: 通过类似C++的多继承的方式, 达到与ruby的模块相同的效果.
    ```py
        class MyMixin1: 
            G1 = 1 # 类变量
            
            @property
            def p1(self):
                return 1

        class MyMixin2: 
            pass
        
        class MyClass(MyMixin1, MyMixin2): 
            pass
        
        c = MyClass()
        c.p1
        MyClass.__mro__ # 祖先链
    ```
* `method missing` 处理: 定义`__getattribute__`方法. 
    ```py
        def __getattribute__(self, name):
            def _missing(*args, **kwargs):
                print "A missing method was called."
                print "The object was %r, the method was %r. " % (self, name)
                print "It was called with %r and %r as arguments" % (args, kwargs)
            return _missing
    ```
* 元类
    * 元类是制造类的工厂. 
    * 可修改类内部(类属性, 类方法). 

    ```py
        class MyMetaClass(type):
            # cls: 类
            # name: 类名
            # bases: 父类列表
            # attrs: 属性/方法字典
            def __new__(cls, name, bases, attrs):
                attrs['name'] = "n"
                attrs['func1'] = lambda self: print("abcd") # 定义实例方法
                return super().__new__(cls, name, bases, attrs)
            
            def f(self): # 这个是类方法, 下面的C类可用
                pass

        class C(metaclass = MyMetaClass): # 指定元类
            pass

        c = C()
        c.func1()
    ```
    * 单例模式
    ```py
        class Singleton(type):
            _instances = {}

            def __call__(cls, *args, **kwargs):
                if cls not in cls._instances:
                    cls._instances[cls] = super().__call__(*args, **kwargs)
                return cls._instances[cls]

        class MyClass(metaclass=Singleton):
            pass

        a = MyClass()
        b = MyClass()
        print(a is b)  # True
    ```
* 类作为对象时的其它方法
    * `__bases__`: 类的所有基类(元组)
    * `__qualname__`: 函数或类的限定名. 
    * `__subclasses__()`: 类的所有直接子类(列表)
    * `mro()`: 超类元组
* 动态类, 动态属性
    * python中**所有类都是`type`类的实例对象**

    ```py
        C = type('C', (object, ), {'num': 0}) # 动态定义类

        # 相当于: 
        class C(object):
            num = 0
        
        # 动态修改对象中的方法(猴子补丁`monkey patch`): 
        import types
        def f(self):
            return self.a
        
        c = C()
        c.f1 = types.MethodType(f, c) # 将上面的f方法作为c对象的f1方法
        setattr(c, "f1", types.MethodType(f, c)) # 也可以这么写, 更灵活

        # 动态访问属性
        hasattr(obj, "k1")
        getattr(obj, "k1") # 若无k1属性, 会出现异常
        setattr(obj, "k1", v1)
        delattr(obj, "k1")
    ```
* 装饰器
    * 固定写法
        ```py
            def my_dec(func): 
                def inner(*args, **kwargs): 
                    '''执行函数前要做的'''
                    ret = func(*args, **kwargs)
                    '''执行函数后要做的'''
                    return ret # 返回的是函数的执行结果
                return inner # 装饰器本身则返回函数对象(其会取代原先的被装饰函数)

            # 使用装饰器
            @my_dec
            def my_func(): 
                pass
        ```
    * 装饰器参数
        ```py
            import inspect

            def hooker(target_func: str):
                def decorator(hook_func: function):
                    hook_func.target_func = target_func
                    return hook_func # 装饰器返回函数对象
                return decorator
            
            @hooker("f1") # 返回一个装饰器
            def hook_f1():
                function_object = globals()["hook_f1"]
                print(function_object.target_func)
        ```
    * 内置装饰器
        * `@classmethod`: 指定类方法. 
            * 被它修饰的方法都有一个`cls`参数, 指向本类. 
            * 一个方法不需要访问对象的实例属性, 但是需要访问到类的类属性, 这个时候就可以考虑把这个方法封装成一个类方法. 
        * `@staticmethod`: 指定静态方法. 
            * 既不需要访问到对象的实例属性, 也不需要访问类的类属性, 这个时候就可以考虑把这个方法封装成一个静态方法. 
        * `@property`: 把一个方法伪装成属性. 
            ```py
                class C: 
                    @property
                    def p1(self): 
                        return _p1

                    @position.setter
                    def p1(self, value):
                        print("Set p1")
                        self._p1 = value
                
                c = C()
                print(c.p1)
                c.p1 = 1
            ```
        * `@wraps`: 说是可以避免下面`example.__name__`返回"wrapper", `example.__doc__`返回"decorator"
            ```py
                from functools import wraps
                def my_decorator(func):
                    def wrapper(*args, **kwargs):
                        '''decorator'''
                        print('Calling decorated function...')
                        return func(*args, **kwargs)
                    return func

                @my_decorator 
                def example():
                    """Docstring""" 
                    print('Called example function')
                print(example.__name__, example.__doc__)
            ```
        * `@cached_property`
            * `from functools import cached_property`
            * 可将修饰的方法作为缓存属性, 方法的返回值会被缓存在实例中. 值仅计算一次. 
    * 类的装饰器: 区别于函数装饰器, 参数是类. 
        ```py
            def entity(cls):
                for key, attr in cls.__dict__.items():
                    ...
            
            @entity
            class C:
                pass
        ```
    * 类作为装饰器
        * 描述符协议: 
            * 实现了`__get__`,` __set__` 和 `__delete__`方法的类(分别用于获取属性值, 设置属性值, 删除属性). 
            * 数据描述符: 同时定义了get/set
            * 非数据描述符: 只定义了get
        * 例
            ```py
                class cached_property:
                    def __init__(self, func): # 被装饰的函数会作为参数传进来
                        self.func = func

                    def __get__(self, instance, owner=None): # `instance`即被装饰方法的`self`参数, 也即方法所属实例. 
                        ...

                    def __set_name__(self, owner, name):
                        ...
            ```
* 命名空间
    * 符号集合; 字典(键为符号名, 值为对象); LEGB法则. 
    * 四类
        * 局部(L): `locals()`
        * 闭包(E): 
            * `<func>.__closure__`
            * `<func>.__closure__[0].cell_contents`
            * 捕获局部变量: 
                ```py
                    from functools import partial

                    def func(a):
                        print(a)

                    num_list = [1, 2, 3]
                    func_list = []
                    for i in num_list: 
                        # func_list.append(lambda: print(i)) # 这样会导致所有lambda用的都是最新的i, 也就是3
                        func_list.append(partial(func, i)) # 这样可以捕获变量i的值

                    func_list[0]() # "1"
                ```

                ```py
                    functions = []
                    for i in range(3):
                        def make_f(i):
                            def f():
                                return i
                            return f
                        functions.append(make_f(i))

                    for f in functions:
                        print(f())  # 输出：0 1 2
                ```
        * 全局(G): `globals()`
        * 内置(B): `dir(__builtins__)`
        * 示例
            ```py
                l1 = 1
                def f():
                    global l1 # 必须有这个声明, 否则报错: local variable 'l1' referenced before assignment
                    l1 += 1
                    l2 = 2
                    def g():
                        nonlocal l2 # 必须有这个声明, 否则报错: local variable 'l2' referenced before assignment
                        l2 += 1
                        return l2
                    return g
                
                g = f()

                # 获取闭包更详细的信息
                import inspect
                inspect.getclosurevars(g)
            ```
    * `reload(<mod>)`: 重新加载`mod`. 
        ```py
            import importlib, os
            os.sys.path.append("<custom_mod.py文件所在路径>")
            cm = importlib.import_module("custom_mod")
            importlib.reload(cm) # 重载模块m

            # 另一种使用模块绝对路径的方法
            import importlib.util
            import sys
            spec = importlib.util.spec_from_file_location("module.name", "/path/to/file.py")
            foo = importlib.util.module_from_spec(spec)
            sys.modules["module.name"] = foo
            spec.loader.exec_module(foo)
            foo.MyClass()
        ```
    * 定义一个无文件的模块: 
        ```py
            import types

            # 创建module对象
            mymodule = types.ModuleType('mymodule', 'This is a custom module')
            mymodule.my_variable = 42

            def greet(name):
                return f"Hello, {name}!"
            mymodule.greet = greet
        ```

# 基本用法
## 文件

### 管道文件
```py
    import os
    tmp_pipe_path = "/tmp/tmp_pipe_" + str(time.time())

    os.mkfifo(tmp_pipe_path)  # 新建管道文件

    tmp_pipe_r = os.open(tmp_pipe_path, os.O_RDONLY|os.O_NONBLOCK) 
    tmp_pipe_w = os.open(tmp_pipe_path, os.O_WRONLY|os.O_NONBLOCK) # 必须先执行上一步打开读管道, 否则会报错提示`No such device or address`

    os.write(tmp_pipe_w, b"hello") # 向管道写入数据
    os.read(tmp_pipe_r, 100) # 从管道读取最大100个字节数据

    ####################################################
    # 方法2: 
    ####################################################
    r, w = os.pipe()	# 创建管道
    # 然后也是调用`os.write`和`os.read`

    ####################################################
    # 方法3: 
    ####################################################
    from multiprocessing import Pipe
    r, w = Pipe()
    w.send("adsfasdf")
    r.recv()
```

## 多进程
* 共享内存
    ```py
        from multiprocessing import shared_memory

        shm_size = 8
        shm = shared_memory.SharedMemory(create=True, size=shm_size)  # 创建共享内存块
        data = shm.buf[:]  # 获取共享内存块的视图
        data[:] = bytes(shm.size)  # 写入全零数据

        # 关闭共享内存块
        # shm.close()
        # shm.unlink()
    ```

# 常用模块
* 参考: 
    * https://blog.csdn.net/qq_40674583/article/details/81940974
    * https://juejin.cn/post/6968320405341732871

## `os`
```py
    os.environ # 一个字典, 包含环境变量

    os.remove() # 删除文件 
    os.unlink() # 删除文件 
    os.rename() # 重命名文件 
    os.listdir() # 列出指定目录下所有文件 
    os.chdir() # 改变当前工作目录
    os.getcwd() # 获取当前工作路径
    os.mkdir() # 新建目录
    os.rmdir() # 删除空目录(删除非空目录, 使用shutil.rmtree() #) #
    os.makedirs() # 递归创建多级目录
    os.removedirs() # 删除多级目录
    os.stat(file) # 获取文件属性
    os.chmod(file) # 修改文件权限
    os.utime(file) # 修改文件时间戳
    os.name(file) # 获取操作系统标识
    os.symlink(source, link_name) # 创建软链接. 注意`source`才是目标文件
    os.system() # 执行操作系统命令
    os.execvp() # 启动一个新进程
    os.fork() # 获取父进程ID, 在子进程返回中返回0
    os.execvp() # 执行外部程序脚本（Uinx）
    os.spawn() # 执行外部程序脚本（Windows）
    os.access(path, mode) # 判断文件权限(详细参考cnblogs) #
    os.wait() # (只在unix有效)等待任何一个子进程结束, 返回一个tuple, 包括子进程的进程ID和退出状态信息：一个16位的数字, 低8位是杀死该子进程的信号编号, 而高8位是退出状态(如果信号编号是0), 其中低8位的最高位如果被置位, 则表示产生了一个core文件。

    # os.path模块：
    os.path.split(filename) # 将文件路径和文件名分割(会将最后一个目录作为文件名而分离) #
    os.path.splitext(filename) # 将文件路径和文件扩展名分割成一个元组
    os.path.dirname(filename) # 返回文件路径的目录部分
    os.path.basename(filename) # 返回文件路径的文件名部分
    os.path.join(dirname,basename) # 将文件路径和文件名凑成完整文件路径
    os.path.abspath(name) # 获得绝对路径
    os.path.splitunc(path) # 把路径分割为挂载点和文件名
    os.path.normpath(path) # 规范path字符串形式
    os.path.exists() # 判断文件或目录是否存在
    os.path.isabs() # 如果path是绝对路径, 返回True
    os.path.realpath(path) # 返回path的真实路径
    os.path.relpath(path[, start]) # 从start开始计算相对路径 
    os.path.normcase(path) # 转换path的大小写和斜杠
    os.path.isdir() # 判断name是不是一个目录, name不是目录就返回false
    os.path.isfile() # 判断name是不是一个文件, 不存在返回false
    os.path.islink() # 判断文件是否连接文件,返回boolean
    os.path.ismount() # 指定路径是否存在且为一个挂载点, 返回boolean
    os.path.samefile() # 是否相同路径的文件, 返回boolean
    os.path.getatime() # 返回最近访问时间 浮点型
    os.path.getmtime() # 返回上一次修改时间 浮点型
    os.path.getctime() # 返回文件创建时间 浮点型
    os.path.getsize() # 返回文件大小 字节单位
    os.path.commonprefix(list) # 返回list(多个路径)中, 所有path共有的最长的路径
    os.path.lexists # 路径存在则返回True,路径损坏也返回True
    os.path.expanduser(path) # 把path中包含的”~”和”~user”转换成用户目录
    os.path.expandvars(path) # 根据环境变量的值替换path中包含的”$name”和”${name}”
    os.path.sameopenfile(fp1, fp2) # 判断fp1和fp2是否指向同一文件
    os.path.samestat(stat1, stat2) # 判断stat tuple stat1和stat2是否指向同一个文件
    os.path.splitdrive(path) # 一般用在windows下, 返回驱动器名和路径组成的元组
    os.path.walk(path, visit, arg) # 遍历path, 给每个path执行一个函数
    os.path.supports_unicode_filenames() # 设置是否支持unicode路径名

    # os.stat
    fileStats = os.stat(path) # 获取到的文件属性列表
    fileStats[stat.ST_MODE] # 获取文件的模式
    fileStats[stat.ST_SIZE] # 文件大小
    fileStats[stat.ST_MTIME] # 文件最后修改时间
    fileStats[stat.ST_ATIME] # 文件最后访问时间
    fileStats[stat.ST_CTIME] # 文件创建时间
    stat.S_ISDIR(fileStats[stat.ST_MODE]) # 是否目录
    stat.S_ISREG(fileStats[stat.ST_MODE]) # 是否一般文件
    stat.S_ISLNK(fileStats[stat.ST_MODE]) # 是否连接文件
    stat.S_ISSOCK(fileStats[stat.ST_MODE]) # 是否COCK文件
    stat.S_ISFIFO(fileStats[stat.ST_MODE]) # 是否命名管道
    stat.S_ISBLK(fileStats[stat.ST_MODE]) # 是否块设备
    stat.S_ISCHR(fileStats[stat.ST_MODE]) # 是否字符设置
```

## `sys`
```py
    sys.argv # 命令行参数List, 第一个元素是程序本身路径 
    sys.path # 返回模块的搜索路径, 初始化时使用PYTHONPATH环境变量的值 
    sys.modules.keys() # 返回所有已经导入的模块列表
    sys.modules # 返回系统导入的模块字段, key是模块名, value是模块 
    sys.exc_info() # 获取当前正在处理的异常类,exc_type、exc_value、exc_traceback当前处理的异常详细信息
    sys.exit(n) # 退出程序, 正常退出时exit(0)
    sys.hexversion # 获取Python解释程序的版本值, 16进制格式如：0x020403F0
    sys.version # 获取Python解释程序的版本信息
    sys.platform # 返回操作系统平台名称
    sys.stdout # 标准输出
    sys.stdout.write('aaa') # 标准输出内容
    sys.stdout.writelines() # 无换行输出
    sys.stdin # 标准输入
    sys.stdin.read() # 输入一行
    sys.stderr # 标准错误输出
    sys.exc_clear() # 用来清除当前线程所出现的当前的或最近的错误信息 
    sys.exec_prefix # 返回平台独立的python文件安装的位置 
    sys.byteorder # 本地字节规则的指示器, big-endian平台的值是‘big‘,little-endian平台的值是‘little‘ 
    sys.copyright # 记录python版权相关的东西 
    sys.api_version # 解释器的C的API版本 
    sys.version_info # ‘final‘表示最终,也有‘candidate‘表示候选, 表示版本级别, 是否有后继的发行 
    sys.getdefaultencoding() # 返回当前你所用的默认的字符编码格式 
    sys.getfilesystemencoding() # 返回将Unicode文件名转换成系统文件名的编码的名字 
    sys.builtin_module_names # Python解释器导入的内建模块列表 
    sys.executable # Python解释程序路径 
    sys.getwindowsversion() # 获取Windows的版本 
    sys.stdin.readline() # 从标准输入读一行, sys.stdout.write(“a”) 屏幕输出a
    sys.setdefaultencoding(name) # 用来设置当前默认的字符编码(详细使用参考文档) 
    sys.displayhook(value) # 如果value非空, 这个函数会把他输出到sys.stdout(详细使用参考文档)
```

## `datetime,date,time`
```py
    # date
    datetime.date.today() # 本地日期对象,(用str函数可得到它的字面表示(2014-03-24))
    datetime.date.isoformat(obj) # 当前[年-月-日]字符串表示(2014-03-24)
    datetime.date.fromtimestamp() # 返回一个日期对象, 参数是时间戳,返回 [年-月-日]
    datetime.date.weekday(obj) # 返回一个日期对象的星期数,周一是0
    datetime.date.isoweekday(obj) # 返回一个日期对象的星期数,周一是1
    datetime.date.isocalendar(obj) # 把日期对象返回一个带有年月日的元组

    # datetime
    datetime.datetime.today() # 返回一个包含本地时间(含微秒数)的datetime对象 2014-03-24 23:31:50.419000
    datetime.datetime.now([tz]) # 返回指定时区的datetime对象 2014-03-24 23:31:50.419000
    datetime.datetime.utcnow() # 返回一个零时区的datetime对象
    datetime.fromtimestamp(timestamp[,tz]) # 按时间戳返回一个datetime对象, 可指定时区,可用于strftime转换为日期表示 
    datetime.utcfromtimestamp(timestamp) # 按时间戳返回一个UTC-datetime对象
    datetime.datetime.strptime('2014-03-16 12:21:21', '%Y-%m-%d %H:%M:%S') # 将字符串转为datetime对象
    datetime.datetime.strftime(datetime.datetime.now(), ‘%Y%m%d %H%M%S‘) # 将datetime对象转换为str表示形式
    datetime.date.today().timetuple() # 转换为时间戳datetime元组对象, 可用于转换时间戳
    datetime.datetime.now().timetuple()

    # time
    time.mktime(timetupleobj) # 将datetime元组对象转为时间戳
    time.time() # 当前时间戳
    time.localtime
    time.gmtime
```

## `hashlib`
```py
    import hashlib

    m = hashlib.md5()  # 相当于建造一个工厂, 用来接收需要加密的数据
    m.update('hello'.encode('utf-8'))  # 接收加密的数据
    m.update('world'.encode('utf-8'))  # 可以多次接收加密的数据
    res = m.hexdigest()   # 对需要加密的数据'helloworld'进行处理, 得到加密后的结果
    print(res)  # fc5e038d38a57032085441e7fe7010b0

    m1 = hashlib.md5('he'.encode('utf-8'))
    m1.update('llo'.encode('utf-8'))
    m1.update('w'.encode('utf-8'))
    m1.update('orld'.encode('utf-8'))
    res = m1.hexdigest()  # # 对需要加密的数据'helloworld'进行处理, 得到加密后的结果
    print(res)  # fc5e038d38a57032085441e7fe7010b0

    # 注意：把一段很长的数据update多次, 与一次update这段长数据, 得到的结果一样。
```

## `random`
```py
    random.random() # 产生0-1的随机浮点数
    random.uniform(a, b) # 产生指定范围内的随机浮点数
    random.randint(a, b) # 产生指定范围内的随机整数
    random.randrange([start], stop[, step]) # 从一个指定步长的集合中产生随机数
    random.choice(sequence) # 从序列中产生一个随机数
    random.shuffle(x[, random]) # 将一个列表中的元素打乱
    random.sample(sequence, k) # 从序列中随机获取指定长度的片断
```

## `string`
```py
    str.capitalize() # 把字符串的第一个字符大写
    str.center(width) # 返回一个原字符串居中, 并使用空格填充到width长度的新字符串
    str.ljust(width) # 返回一个原字符串左对齐, 用空格填充到指定长度的新字符串
    str.rjust(width) # 返回一个原字符串右对齐, 用空格填充到指定长度的新字符串
    str.zfill(width) # 返回字符串右对齐, 前面用0填充到指定长度的新字符串
    str.count(str,[beg,len]) # 返回子字符串在原字符串出现次数, beg,len是范围
    str.decode(encodeing[,replace]) # 解码string,出错引发ValueError异常
    str.encode(encodeing[,replace]) # 解码string
    str.endswith(substr[,beg,end]) # 字符串是否以substr结束, beg,end是范围
    str.startswith(substr[,beg,end]) # 字符串是否以substr开头(substr可以是正则表达式(r"sth")), beg, end是范围 
    str.expandtabs(tabsize = 8) # 把字符串的tab转为空格, 默认为8个
    str.find(str,[stat,end]) # 查找子字符串在字符串第一次出现的位置, 否则返回-1
    str.index(str,[beg,end]) # 查找子字符串在指定字符中的位置, 不存在报异常
    str.isalnum() # 检查字符串是否以字母和数字组成, 是返回true否则False
    str.isalpha() # 检查字符串是否以纯字母组成, 是返回true,否则false
    str.isdecimal() # 检查字符串是否以纯十进制数字组成, 返回布尔值
    str.isdigit() # 检查字符串是否以纯数字组成, 返回布尔值
    str.islower() # 检查字符串是否全是小写, 返回布尔值
    str.isupper() # 检查字符串是否全是大写, 返回布尔值
    str.isnumeric() # 检查字符串是否只包含数字字符, 返回布尔值
    str.isspace() # 如果str中只包含空格, 则返回true,否则FALSE
    str.title() # 返回标题化的字符串（所有单词首字母大写, 其余小写）
    str.istitle() # 如果字符串是标题化的(参见title())则返回true,否则false
    str.join(seq) # 以str作为连接符, 将一个序列中的元素连接成字符串
    str.split(str='', num) # 以str作为分隔符, 将一个字符串分隔成一个序列, num是最大分割次数
    str.splitlines(num) # 以行分隔, 返回各行内容作为元素的列表
    str.lower() # 将大写转为小写
    str.upper() # 转换字符串的小写为大写
    str.swapcase() # 翻换字符串的大小写
    str.lstrip() # 去掉字符左边的空格和回车换行符
    str.rstrip() # 去掉字符右边的空格和回车换行符
    str.strip() # 去掉字符两边的空格和回车换行符
    str.partition(substr) # 从substr出现的第一个位置起, 将str分割成一个3元组。
    str.replace(str1,str2,num) # 查找str1替换成str2, num是替换次数
    str.rfind(str[,beg,end]) # 从右边开始查询子字符串
    str.rindex(str,[beg,end]) # 从右边开始查找子字符串位置 
    str.rpartition(str) # 类似partition函数, 不过从右边开始查找
    str.translate(str,del=‘‘) # 按str给出的表转换string的字符, del是要过虑的字符
```

## `urllib`
```py
    urllib.quote(string[,safe]) # 对字符串进行编码。参数safe指定了不需要编码的字符
    urllib.unquote(string) # 对字符串进行解码
    urllib.quote_plus(string[,safe]) # 与urllib.quote类似, 但这个方法用‘+‘来替换‘ ‘, 而quote用‘%20‘来代替‘ ‘
    urllib.unquote_plus(string ) # 对字符串进行解码
    urllib.urlencode(query[,doseq]) # 将dict或者包含两个元素的元组列表转换成url参数。例如, 字典{‘name‘:‘wklken‘,‘pwd‘:‘123‘}将被转换为”name=wklken&pwd=123″
    urllib.pathname2url(path) # 将本地路径转换成url路径
    urllib.url2pathname(path) # 将url路径转换成本地路径
    urllib.urlretrieve(url[,filename[,reporthook[,data]]]) # 下载远程数据到本地
        # filename: 指定保存到本地的路径（若未指定该, urllib生成一个临时文件保存数据）
        # reporthook: 回调函数, 当连接上服务器、以及相应的数据块传输完毕的时候会触发该回调
        # data: 指post到服务器的数据

    urllrs = urllib.urlopen(url[,data[,proxies]]) # 抓取网页信息, [data]post数据到Url,proxies设置的代理
    urlrs.readline() # 跟文件对象使用一样
    urlrs.readlines() # 跟文件对象使用一样
    urlrs.fileno() # 跟文件对象使用一样
    urlrs.close() # 跟文件对象使用一样
    urlrs.info() # 返回一个httplib.HTTPMessage对象, 表示远程服务器返回的头信息
    urlrs.getcode() # 获取请求返回状态HTTP状态码
    urlrs.geturl() # 返回请求的URL
```

## `re`
* 常用正则表达式符号和语法：
    * '.' 匹配所有字符串, 除\n以外
    * ‘-’ 表示范围[0-9]
    * '*' 匹配前面的子表达式零次或多次。要匹配 * 字符, 请使用 \*。
    * '+' 匹配前面的子表达式一次或多次。要匹配 + 字符, 请使用 \+
    * '^' 匹配字符串开头
    * ‘$’ 匹配字符串结尾 re
    * '\' 转义字符,  使后一个字符改变原来的意思, 如果字符串中有字符*需要匹配, 可以\*或者字符集[*] re.findall(r'3\*','3*ds')结['3*']
    * '*' 匹配前面的字符0次或多次 re.findall("ab*","cabc3abcbbac")结果：['ab', 'ab', 'a']
    * ‘?’ 匹配前一个字符串0次或1次 re.findall('ab?','abcabcabcadf')结果['ab', 'ab', 'ab', 'a']
    * '{m}' 匹配前一个字符m次 re.findall('cb{1}','bchbchcbfbcbb')结果['cb', 'cb']
    * '{n,m}' 匹配前一个字符n到m次 re.findall('cb{2,3}','bchbchcbfbcbb')结果['cbb']
    * '\d' 匹配数字, 等于[0-9] re.findall('\d','电话:10086')结果['1', '0', '0', '8', '6']
    * '\D' 匹配非数字, 等于[^0-9] re.findall('\D','电话:10086')结果['电', '话', ':']
    * '\w' 匹配字母和数字, 等于[A-Za-z0-9] re.findall('\w','alex123,./;;;')结果['a', 'l', 'e', 'x', '1', '2', '3']
    * '\W' 匹配非英文字母和数字,等于[^A-Za-z0-9] re.findall('\W','alex123,./;;;')结果[',', '.', '/', ';', ';', ';']
    * '\s' 匹配空白字符 re.findall('\s','3*ds \t\n')结果[' ', '\t', '\n']
    * '\S' 匹配非空白字符 re.findall('\s','3*ds \t\n')结果['3', '*', 'd', 's']
    * '\A' 匹配字符串开头
    * '\Z' 匹配字符串结尾
    * '\b' 匹配单词的词首和词尾, 单词被定义为一个字母数字序列, 因此词尾是用空白符或非字母数字符来表示的
    * '\B' 与\b相反, 只在当前位置不在单词边界时匹配
    * '(?P<name>...)' 分组, 除了原有编号外在指定一个额外的别名 re.search("(?P<province>[0-9]{4})(?P<city>[0-9]{2})(?P<birthday>[0-9]{8})","371481199306143242").groupdict("city") 结果{'province': '3714', 'city': '81', 'birthday': '19930614'}
    * [] 是定义匹配的字符范围。比如 [a-zA-Z0-9] 表示相应位置的字符要匹配英文字符和数字。[\s*]表示空格或者*号。

```py
    m = re.match(pattern, string, flags=0) # 从字符串的起始位置匹配, 如果起始位置匹配不成功的话, match()就返回none
    m.group(0) # 获取匹配的字符串
    m.group(1) # 获取捕获的第一个分组

    re.search(pattern, string, flags=0) # 扫描整个字符串并返回第一个成功的匹配
    re.findall(pattern, string, flags=0) # 找到RE匹配的所有字符串, 并把他们作为一个列表返回(若正则表达式中有多个圆括号, 会将每个圆括号匹配的字符串放到一个元组中, 而返回的列表由这些元组组成)
    re.finditer(pattern, string, flags=0) # 找到RE匹配的所有字符串, 并把他们作为一个迭代器返回(可获得每个匹配字符串及匹配的起始结束位置)
    re.sub(pattern, repl, string, count=0, flags=0) # 替换匹配到的字符串
    re.split(pattern: str | Pattern[str], string: str, maxsplit: int = 0, flags: _FlagsType = 0)  # 使用正则表达式分割字符串

    # `finditer`示例: 匹配"r", "12323", "22"
    iter = re.finditer(r"\s*([^\s]+)\s*", " r   12323  22 ")
    for match_obj in iter:
        print(repr(match_obj[1]))

    # `findall`示例
    re.findall(r"\s*([^\s]+),\s*([^\s]+)\s*", " r, 1   s,2  t,  3 ") #=> [('r', '1'), ('s', '2'), ('t', '3')]

    # `sub`示例: 使用`\1`表示捕获的第一个分组
    re.sub("(<|>)", r"\\\1", "<a> [<b> (64)]")
```

## `struct`
* 字节序等: 
|字符|字节顺序|大小|对齐方式|
|-|-|-|-|
|@|按原字节|按原字节|按原字节(**会因按系统对齐而产生多余的字节**)|
|=|按原字节|标准|标准(**不会有多余的字节**)|
|<|小端|标准|无|
|>|大端|标准|无|
|!|网络(大端)|标准|无|

* 格式
|C 类型 |Python 类型 |标准大小 |备注 |
|-|-|-|-|
|`x`|填充字节|-|(7)|
|`c`|`char`|长度为 1 的字节串 |1|
|`b`|`signed char`|integer|1|(1), (2)|
|`B`|`unsigned char`|integer|1|(2)|
|`?`|`_Bool`|bool|1|(1)|
|`h`|`short`|integer|2|(2)|
|`H`|`unsigned short`|integer|2|(2)|
|`i`|`int`|integer|4|(2)|
|`I`|`unsigned int`|integer|4|(2)|
|`l`|`long`|integer|4|(2)|
|`L`|`unsigned long`|integer|4|(2)|
|`q`|`long long`|integer|8|(2)|
|`Q`|`unsigned long long`|integer|8|(2)|
|`n`|`ssize_t`|integer|-|(3)|
|`N`|`size_t`|integer|-|(3)|
|`e`|-|float|2|(4)|
|`f`|`float`|float|4|(4)|
|`d`|`double`|float|8|(4)|
|`s`|`char[]`|字节串|-|(9)|
|`p`|`char[]`|字节串|-|(8)|
|`P`|`void*`|integer|-|(5)|

```py
    import struct
    
    struct.pack('>h', 1023) # 数据转字节串 #=> b'\x03\xff'

    struct.unpack('>bhl', b'\x01\x00\x02\x00\x00\x00\x03') # 字节串转元组 #=> (1, 2, 3)
```



## `math`
* `ceil`: 取大于等于x的最小的整数值, 如果x是一个整数, 则返回x
* `copysign`: 把y的正负号加到x前面, 可以使用0
* `cos`: 求x的余弦, x必须是弧度
* `degrees`: 把x从弧度转换成角度
* `e`: 常量e
* `exp`: 返回math.e,也就是2.71828的x次方
* `expm1`: 返回math.e的x(其值为2.71828)次方的值减１
* `fabs`: 返回x的绝对值
* `factorial`: 取x的阶乘的值
* `floor`: 取小于等于x的最大的整数值, 如果x是一个整数, 则返回自身
* `fmod`: 得到x/y的余数, 其值是一个浮点数
* `frexp`: 返回一个元组(m,e),其计算方式为：x分别除0.5和1,得到一个值的范围
* `fsum`: 对迭代器里的每个元素进行求和操作
* `gcd`: 返回x和y的最大公约数
* `hypot`: 如果x是不是无穷大的数字,则返回True,否则返回False
* `isfinite`: 如果x是正无穷大或负无穷大, 则返回True,否则返回False
* `isinf`: 如果x是正无穷大或负无穷大, 则返回True,否则返回False
* `isnan`: 如果x不是数字True,否则返回False
* `ldexp`: 返回x*(2**i)的值
* `log`: 返回x的自然对数, 默认以e为基数, base参数给定时, 将x的对数返回给定的base,计算式为：log( x)/log(base)
* `log10`: 返回x的以10为底的对数
* `log1p`: 返回x+1的自然对数(基数为e)的值
* `log2`: 返回x的基2对数
* `modf`: 返回由x的小数部分和整数部分组成的元组
* `pi`: 数字常量, 圆周率
* `pow`: 返回x的y次方, 即x**y
* `radians`: 把角度x转换成弧度
* `sin`: 求x(x为弧度)的正弦值
* `sqrt`: 求x的平方根
* `tan`: 返回x(x为弧度)的正切值
* `trunc`: 返回x的整数部分

## `logging`
```py
    import logging

    # 数字大小代表不同等级
    CRITICAL = 50 
    ERROR = 40
    WARNING = 30 
    INFO = 20
    DEBUG = 10
    NOTSET = 0 


    logging.debug('调试debug')
    logging.info('消息info')
    logging.warning('警告warn')
    logging.error('错误error')
    logging.critical('严重critical')

    # WARNING:root:警告warn
    # ERROR:root:错误error
    # CRITICAL:root:严重critical

    logging.basicConfig(filename='test.log',
                        format='%(asctime)s - %(name)s - %(levelname)s -%(module)s:  %(message)s',
                        datefmt='%Y-%m-%d %H:%M:%S %p',
                        level=10)

    # 上述参数及其他未用到的参数介绍如下：
    # filename：用指定的文件名创建FiledHandler, 用来打印到文件中
    # datefmt：指定日期时间格式。
    # level：根据数字设置的日志级别

    # format参数中可能用到的格式化串：
    # %(name)s Logger的名字
    # %(levelno)s 数字形式的日志级别
    # %(levelname)s 文本形式的日志级别
    # %(pathname)s 调用日志输出函数的模块的完整路径名, 可能没有
    # %(filename)s 调用日志输出函数的模块的文件名
    # %(module)s 调用日志输出函数的模块名
    # %(funcName)s 调用日志输出函数的函数名
    # %(lineno)d 调用日志输出函数的语句所在的代码行
    # %(asctime)s 字符串形式的当前时间。默认格式是 “2003-07-08 16:49:45,896”。逗号后面的是毫秒
    # %(thread)d 线程ID。可能没有
    # %(threadName)s 线程名。可能没有
    # %(process)d 进程ID。可能没有
    # %(message)s用户输出的消息

```

### 项目开发中的用法
```py
    # settings.py

    # 日志文件
    LOG_PATH = os.path.join(BASE_DIR,'log')
    LOG_UER_PATH = os.path.join(LOG_PATH,'access.log')

    # 定义三种日志输出格式 
    standard_format = '%(asctime)s - %(threadName)s:%(thread)d - 日志名字:%(name)s - %(filename)s:%(lineno)d -' \
                    '%(levelname)s - %(message)s'

    simple_format = '[%(levelname)s][%(asctime)s][%(filename)s:%(lineno)d]%(message)s'

    test_format = '%(asctime)s] %(message)s'

    # 日志配置字典
    LOGGING_DIC = {
        'version': 1,
        'disable_existing_loggers': False,
        'formatters': {
            'standard': {
                'format': standard_format
            },
            'simple': {
                'format': simple_format
            },
            'test': {
                'format': test_format
            },
        },
        'filters': {},
        # handlers是日志的接收者, 不同的handler会将日志输出到不同的位置
        'handlers': {
            #打印到终端的日志
            'console': {
                'level': 'DEBUG',
                'class': 'logging.StreamHandler',  # 打印到屏幕
                'formatter': 'simple'
            },
            'default': {
                'level': 'DEBUG',
                'class': 'logging.handlers.RotatingFileHandler',  # 保存到文件
                # 'maxBytes': 1024*1024*5,  # 日志大小 5M
                # 'maxBytes': 1000,
                # 'backupCount': 5,
                'filename': LOG_UER_PATH,  # os.path.join(os.path.dirname(os.path.dirname(__file__)),'log','a2.log')
                'encoding': 'utf-8',
                'formatter': 'standard',

            },
            #打印到文件的日志,收集info及以上的日志
            'other': {
                'level': 'DEBUG',
                'class': 'logging.FileHandler',  # 保存到文件
                'filename': LOG_UER_PATH, # os.path.join(os.path.dirname(os.path.dirname(__file__)),'log','a2.log')
                'encoding': 'utf-8',
                'formatter': 'test',

            },
        },
        
        # loggers是日志的产生者, 产生的日志会传递给handler然后控制输出
        'loggers': {
            # logging.getLogger(__name__)拿到的logger配置, 如果我们想要不同logger名的logger对象都共用一段配置, 那么肯定不能在loggers子字典中定义n个key , 解决方式就是定义一个空的key,这样我们再取logger对象时logging.getLogger(__name__), 不同的文件__name__不同, 这保证了打印日志时标识信息不同, 但是拿着该名字去loggers里找key名时却发现找不到, 于是默认使用key=''的配置
            '': {
                'handlers': ['default', ],  # 这里把上面定义的两个handler都加上, 即log数据既写入文件又打印到屏幕
                'level': 'DEBUG',  # loggers(第一层日志级别关限制)--->handlers(第二层日志级别关卡限制)
                'propagate': False,  # 默认为True, 向上（更高level的logger）传递, 通常设置为False即可, 否则会一份日志向上层层传递
            },
        },
    }

```

```py
    # test.py

    # logging是一个包, 需要使用其下的config、getLogger
    from logging import config
    from logging import getLogger

    # 加载日志字典配置
    logging.config.dictConfig(settings.LOGGING_DIC)

    # 生成日志对象
    logger = logging.getLogger('test')

    # 记录日志
    logger.info('记录一条日志')
```

## `.`
```py

```

## 其它
* `atexit.register(fun,args,args2..)`: 注册函数func, 在解析器退出前调用该函数

## `SQLAlchemy`
* 参考: https://www.cnblogs.com/jiangxiaobo/p/12360954.html
```py
    #coding:utf-8

    from sqlalchemy import create_engine
    from sqlalchemy.ext.declarative import declarative_base
    from sqlalchemy import Column, String, Integer
    import threading

    engine = create_engine(
        "mysql+pymysql://root@127.0.0.1:3306/learningsql?charset=utf8",
        max_overflow = 0,  #超过连接池大小外最多创建的连接, 为0表示超过5个连接后, 其他连接请求会阻塞 （默认为10）
        pool_size = 5,    #连接池大小（默认为5）
        pool_timeout = 30,  #连接线程池中, 没有连接时最多等待的时间, 不设置无连接时直接报错 （默认为30）
        pool_recycle = -1  #多久之后对线程池中的线程进行一次连接的回收（重置） （默认为-1）
    )
                
    def task():
        cur =engine.execute("select * from t_sales where id>%s",(2,))  #engine直接执行
        result = cur.fetchone() # 取一行
        
        for i in cur: # 遍历
            print i.id, i.cost, i.ctime,i.desc
            
        cur.close()
        print(result)

    if __name__=="__main__":
        for i in range(10):
            t = threading.Thread(target=task)
            t.start()

```

### ORM
```py
    import datetime
    from sqlalchemy import create_engine
    from sqlalchemy.ext.declarative import declarative_base
    from sqlalchemy import Column, String, Integer, DateTime, Text

    Base = declarative_base()

    class User(Base):
        __tablename__="users"
        id = Column(Integer,primary_key=True)
        name = Column(String(32),index=True, nullable=False) #创建索引, 不为空
        email = Column(String(32),unique=True)
        ctime = Column(DateTime, default = datetime.datetime.now)  #传入方法名datetime.datetime.now
        extra = Column(Text,nullable=True)  
        
        __table_args__ = {
            UniqueConstraint('id', 'name', name='uix_id_name'), #设置联合唯一约束
            Index('ix_id_name', 'name', 'email'),               # 创建索引
        }

    Base.metadata.create_all(engine)   #创建所有定义的表
    Base.metadata.drop_all(engine)   #删除所有创建的表

    # 增删查改
    #############################################################
    Session = sessionmaker(bind=engine)

    #每次进行数据库操作时都要创建session
    session = Session()

    #*****************增加数据********************
    pur_order = PurchaseOrder(cost=19.7, desc="python编程之路")

    session.add(pur_order)
    session.add_all(
                [PurchaseOrder(cost=39.7,desc="linux操作系统"),
                PurchaseOrder(cost=59.6,desc="python cookbook")])
    session.commit()

    #*****************修改数据********************

    session.query(PurchaseOrder).filter(PurchaseOrder.id>2).update({"cost":29.7})
    session.query(PurchaseOrder).filter(PurchaseOrder.id==2).update({"cost":PurchaseOrder.cost+40.1},synchronize_session=False)  #synchronize_session用于query在进行delete or update操作时, 对session的同步策略。

    # 方法二
    update(User).where(User.id == 1).values(name='New Name')

    # 方法三
    session.merge(User(id=1, name='New Name'))

    # 方法四
    session.replace(User(id=1, name='New Name'))

    session.commit()


    #*****************删除数据********************
    session.query(PurchaseOrder).filter(PurchaseOrder.id==1).delete()
    session.commit()

    #*****************查询数据********************
    ret = session.query(PurchaseOrder).all()
    ret = session.query(PurchaseOrder).filter(PurchaseOrder.id==2).all()  #包含对象的列表
    ret = session.query(PurchaseOrder).filter(PurchaseOrder.id==2).first() #单个对象
    ret = session.query(PurchaseOrder).filter_by(id=2).all()  #通过列名字的表达式
    ret = session.query(PurchaseOrder).filter_by(id=2).first()
    ret = session.query(PurchaseOrder).filter(text("id<:value and cost>:price")).params(value=6,price=15).order_by(PurchaseOrder.cost).all()
    ret = session.query(PurchaseOrder).from_statement(text("SELECT * FROM purchase_order WHERE cost>:price")).params(price=40).all()
    print ret
    for i in ret:
        print i.id, i.cost, i.ctime,i.desc
        
    ret2 = session.query(PurchaseOrder.id,PurchaseOrder.cost.label('totalcost')).all()  #只查询两列, ret2为列表
    #print ret2

    #关闭session
    session.close()
```
* `alembic`: 用来做OMR模型与数据库的迁移与映射. 需要用pip单独安装. 
    * `init <仓库名>`: 生成一个`<仓库名>`目录和一个`alembic.ini`文件. 
        * `alembic.ini`: 
            * 修改`sqlalchemy.url`, 赋值为目标数据库的连接url. 
        * ``<仓库名>/``: 
            * `env.py`: 修改`target_metadata`参数, 值为前面的`Base`模块的`Base.metadata`
    * `revision --autogenerate -m "注释"`: 创建迁移文件
    * `upgrade head|<版本>`: 迁移到数据库
    * `downgrade head|<版本>`: 降级
    * `history`: 列出迁移版本及信息
## fastapi
* 问题
    * 
        * 参考: [解决fastapi访问/docs和/redoc接口文档显示空白或无法加载](https://blog.csdn.net/weixin_43936332/article/details/131020726)
# PyQt5
* 安装: 用pip安装`PyQt5`和`qt5-tools`. 目前只支持到`python 3.9`(20240619)
* 工具: 
    * designer: 运行`qt5-tools designer`
        * 编辑信号槽: 参考[Qt Designer: how to add custom slot and code to a button](https://stackoverflow.com/questions/7964869/qt-designer-how-to-add-custom-slot-and-code-to-a-button)
        * 控件提升: 参考 [自定义控件与提升控件](https://blog.csdn.net/yuanzhapi2090/article/details/130205399)
    * ui文件转py文件
        * `python -m PyQt5.uic.pyuic $FileName$ -o $FileNameWithoutExtension$.py`
    * 发布: `pyinstaller -F my_gui.py`
* 主程序
    ```py
        from PyQt5 import QtWidgets, uic
        from PyQt5.QtWidgets import QApplication
        import sys
        from my_gui import Ui_MainWindow

        class MyApp(QMainWindow): 
            def __init__(self):
                super().__init__()

                # 方法1, 创建Ui_MainWindow对象(Ui_MainWindow是使用pyuic转换ui文件生成的py文件中的类), 然后调用其setupUI方法, 传入本类的实例(即self)作为参数
                self.ui = Ui_MainWindow()
                self.ui.setupUi(self)

                # uic.loadUi('maindlg.ui', self) # 方法2, 直接载入ui文件
        
        if __name__ == "__main__":
            app = QApplication(sys.argv)
            myapp = MyApp()
            myapp.show()
            sys.exit(app.exec_())
    ```
* 自定义信号
    ```py
        class MyData(QObject):
            valueChanged = pyqtSignal(str) # 定义信号
            # valueChanged = pyqtSignal(str, int) # 信号的参数是一个str类型数值和一个int类型数值
            # valueChanged = pyqtSignal([str], [str, int]) # 信号的参数是一个str类型数值, 或者一个str类型数值和一个int类型数值

            def __init__(self, value):
                super().__init__()
                self._value = value

            @property
            def value(self):
                return self._value

            @value.setter
            def value(self, new_value):
                if self._value != new_value:
                    self._value = new_value
                    self.valueChanged.emit(new_value) # 变量发生变化时发射信号

        
        # 为信号绑定槽函数
        MyData("test").valueChanged.connect(lambda s: print(f"new value: {s}"))
    ```
    * 错误
        * `'PyQt5.QtCore.pyqtSignal' object has no attribute 'connect'`
            * 注意**信号一定要定义为类变量**

* 简单示例程序(参考: https://mp.weixin.qq.com/s?__biz=MzU0MDQ1NjAzNg==&mid=2247526677&idx=3&sn=00e809bfb65e3d43c8c0fcb5e0bc720f&chksm=fb3acc1ecc4d4508c6fc3d23349da6525385f1011358f0e682057be1475569152b6db937defd&scene=27)
    ```py
        import sys
        from PyQt5.QtWidgets import QApplication, QMainWindow, QLabel
        app = QApplication(sys.argv) # 每个GUI都必须包含一个Qapplication, argv表示获取命令行参数, 如果不用获取, 则可以使用[]代替. 
        win = QMainWindow()
        win.setGeometry(400, 400, 400, 300)
        win.setWindowTitle("Pyqt5 Tutorial")
        win.show()
        sys.exit(app.exec_()) # 设置窗口一直运行指导使用关闭按钮进行关闭
    ```