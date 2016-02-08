# 执行环境与作用域
1. **执行环境（`execution context`），简称为环境，别称：执行上下文**；

2. 执行环境定义了变量或函数有权访问的其他数据，决定了它们各自的行为；

3. 变量对象：`JS` **解析器层面概念**，代码层面无法访问，解析器处理数据时后台使用
    >每个**执行环境**都有一个与之关联的**变量对象**，**环境中定义的所有变量和函数都保存在这个对象中**。

4. 执行环境具有分层机制，**全局执行环境** 是最外围的一个执行环境，在 `Web` 浏览器中，全局执行环境是 **`window`** 对象，所有 **全局变量** 和函数都是做为 window 对象的属性和方法创建的；
    ```javascript
    var foo = "bar";
    alert(window.foo); // "bar"

    function addSum(num1, num2) {
        return (num1 + num2);
    }
    window.addSum(1, 1); // 2
    ```
5. 每个函数也有自己的**执行环境**，某个执行环境中的 **所有代码执行完毕后**，该环境被 **销毁**，保存在其中的所有变量和函数定义也随之被销毁；

6. 全局环境直到应用程序退出 -- 例如**关闭网页**或**浏览器**时才会被销毁；

7. **执行流**
    > `JS` 脚本代码流式执行，当 **执行流进入一个函数**，函数的 **环境就会被推入一个环境栈** -- 由各个层级环境组成的栈，此时 **程序的控制权在该函数手中**；函数执行完后，栈将其环境弹出，把控制权返回给之前的执行环境

8. 作用域链：一个函数，活动对象最开始时只包含一个变量，即 `arguments` 对象；作用域链中的下一个 **变量对象** 来自 **包含（外部）环境**，再下一个变量对象则来自下一个包含环境。这样一直延续到全局执行函数。
    > 代码在一个执行环境中执行时，会创建变量对象（见 *No.3*）的一个作用域链：**作用域链是针对变量对象来说的**
<br>
    > **作用域链的用途**：保证对 **执行环境有权访问**（前提条件） 的所有变量和函数的**有序访问**（从作用域链下端向上端访问）
<br>
> **作用域链的前端**：始终是 当前执行的代码 所在环境的 **变量对象**；如果环境是函数，则将其 **活动对象** （activation object）作为 **变量对象**
<br>
> **作用域链的最后端**：全局执行环境的变量对象始终是作用域链中的最后一个对象

9. 作用域链的一个实例：
    ```javascript
    var color =  "blue";

    function changeColor() {
        var anotherColor = "red";

        function swapColors() {
            var tempColor = anotherColor;
            anotherColor = color;
            color = tempColor;
        }

        swapColors()
    }
    ```
    作用域链图示：矩形表示特定的执行环境
    
    ![2016-02-07_152019.jpg-18.4kB][1]
    上述代码设计 3 个执行环境：全局环境、`changeColor()` 局部环境、`swapColors()` 局部环境
    * 全局环境的变量对象由 **`color` 变量、`changeColors()` 函数** 组成
    * `changeColors()` 局部环境的变量对象由 `anotherColor` 变量、`swapColors()` 函数组成
    * `swapColors()` 局部环境的变量对象由 `tempColor` 变量单独构成

10. **标识符解析** -- 上级回溯机制：沿着作用域链一级一级有序地搜索标识符的过程；从作用域链的前端开始，逐级后溯，直到找到标识符（无法找到标识符，通常会报错）；

11. 函数参数也被当作变量来对待，访问规格与函数执行环境中的其他规则相同。

## 延长作用域链
1. **增长作用域链的机制**：有些语句可以在作用域链的前端 **临时增加** 一个变量对象，该变量对象在代码执行后被移除；

2. 增长作用域链的两种方法
    当执行流进入下列任何一个语句时，作用域链就会得到加长

    * `try-catch` 语句的 `catch` 块
    * `with` 语句
3. 实例：
    * `with` 语句
    ```javascript
    function buildURL() {
        var qs = '?debug=true';

        with(location) { // 临时增加 location 变量对象
            var url = href + qs
        } // 代码执行完毕后 location 移除，但 url 新值保留

        return url
    }

    buildURL();
    ```
    * `catch` 语句
    ```javascript
    function message() {
        try {
            adddlert("Welcome guest!");
        }
        catch(err) { // 创建引得变量对象 err -- 抛出的错误对象
            txt="There was an error on this page.\n\n";
            txt+="Error description: " + err.message + "\n\n";
            txt+="Click OK to continue.\n\n";
            alert(txt);
        }
    }

    message(); // ...adddlert is not defined...
    ```
4. **冷知识**：IE 8 下 bug -- catch 语句中捕获的错误对象会被添加到执行环境的变量对象，而不是 catch 语句的变量对象中；换句话说：**即使在 catch 块的外部也可以访问到错误对象**

5. `ES` 没有 **块级作用域**
    > 在 `C` `C++` `Java` 等语言中，不同于 `JS` 的执行环境，它们存在的是 **块级作用域**

6. 由 `for` 语句创建的变量 `i` 即使在 `for` 循环执行结束后，也依旧会存在于循环外部的执行环境中；

7. 使用 `var` 声明的变量会自动被添加到最接近的执行环境中；初始化变量没有使用 `var` 声明，该变量会自动被添加到全局环境；

8. **最佳实践**：初始化变量前一定要先声明
    >不初始化即声明变量造成的问题：
    * 全局污染；
    * 严格模式下报错。

9. 查询标识符：当在某个环境中为了读取或写入而引用一个标识符时，会从从作用域链前端开始搜索，逐级上溯，直到找到为止；搜索的最后一级是全局环境的变量对象，如仍旧未能搜索到，说明变量未声明，报错；

10. 变量搜索要付出性能代价，访问局部变量比访问全局变量要快；随着 `JavaScript` 引擎的优化，这个差别已经很小；

  [1]: http://static.zybuluo.com/yangfch3/1wz39jjosri2tur0sh1wd3nj/2016-02-07_152019.jpg