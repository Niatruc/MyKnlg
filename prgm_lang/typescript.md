* 参考
    * https://www.runoob.com/typescript/ts-tutorial.html

* 安装: `npm install -g typescript`
* `tsc`
    * `tsc app.ts`: 将在当前目录下生成`app.js`
    * 常用参数: 
        |参数|编译参数说明|
        |-|-|
        |`--help`|显示帮助信息|
        |`--module`|载入扩展模块|
        |`--target`|设置 ECMA 版本|
        |`--declaration`|额外生成一个 `.d.ts` 扩展名的文件. `tsc ts-hw.ts `--declaration``: 会生成 `ts-hw.d.ts`, `ts-hw.js` 两个文件. |
        |`--removeComments`|删除文件的注释|
        |`--out`|编译多个文件并合并到一个输出的文件|
        |`--sourcemap`|生成一个 `sourcemap (.map)` 文件`.sourcemap`是一个存储源代码与编译代码对应位置映射的信息文件. |
        |`--module noImplicitAny`|在表达式和声明上有隐含的 `any` 类型时报错|
        |`--watch`|在监视模式下运行编译器. 会监视输出文件, 在它们改变时重新编译. |
* 相比js新增的功能
    * 类型批注和编译时类型检查: 
    * 类型推断: 
    * 类型擦除: 
    * 接口: 
    * 枚举: 
    * Mixin: 
    * 泛型编程: 
    * 名字空间: 
    * 元组: 
    * Await: 

* 数据类型


    * 任意类型`any`
        * 声明为 `any` 的变量可以赋予任意类型的值. 
    * 数字类型`number`
        * 双精度 64 位浮点值, 用来表示整数和分数. 
            ```ts
                let binaryLiteral: number = 0b1010; // 二进制
                let octalLiteral: number = 0o744;    // 八进制
                let decLiteral: number = 6;    // 十进制
                let hexLiteral: number = 0xf00d;    // 十六进制
            ```
    * 字符串类型`string`
        * 使用单引号(')或双引号(")来表示字符串类型. 反引号(`)来定义多行文本和内嵌表达式. 
            ```ts
                let name: string = "Runoob";
                let years: number = 5;
                let words: string = `您好, 今年是 ${ name } 发布 ${ years + 1} 周年`;
            ```
    * 布尔类型`boolean`: `true` 和 `false`. 
    * 数组类型
        ```ts
            // 在元素类型后面加上[]
            let arr: number[] = [1, 2];

            // 或者使用数组泛型
            let arr: Array<number> = [1, 2];
        ```
    * 元组
        ```ts
            let x: [string, number];
            x = ['Runoob', 1];    // 运行正常
            x = [1, 'Runoob'];    // 报错
            console.log(x[0]);    // 输出 Runoob
        ```
    * 枚举`enum`: 
        ```ts
            enum Color {Red, Green, Blue};
            let c: Color = Color.Blue;
            console.log(c);    // 输出 2
        ```
    * `void`: 用于标识方法返回值的类型, 表示该方法没有返回值. 
        ```ts
            function hello(): void {
                alert("Hello Runoob");
            }
        ```
    * `null`: 表示对象值缺失. 
    * `undefined`: 用于初始化变量为一个未定义的值. 
    * `never`: never 是其它类型(包括 null 和 undefined)的子类型, 代表从不会出现的值. 