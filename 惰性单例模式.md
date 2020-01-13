# 惰性单例模式
之前介绍了JS中类的单例模式，这次我们讨论下单例模式的应用。在众多网站中，登录框的实现方式就是一个单例，点击一次就展示一次，所以我们可以在页面加载好的时候就创建一个登录框，点击页面上的登录按钮时，用于控制它的显示和隐藏。

![](https://img2018.cnblogs.com/blog/1852401/202001/1852401-20200113113209045-572876576.png)

代码实现：

![](https://img2018.cnblogs.com/blog/1852401/202001/1852401-20200113113924306-225800117.png)

<!--
```html
<body>
    <button id="loginbtn">登录</button>
    <script>
        const btn = document.querySelector("#loginbtn");
        const loginDiv = (function () {
            const div = document.createElement("div");
            div.innerHTML = "登录浮窗";
            div.style.display = "none";
            document.body.appendChild(div);
            return div;
        })();
        btn.onclick = function () {
            loginDiv.style.display = block;
        }
    </script>
</body>
```
-->

这样的问题就是，如果用户进来后所有的操作根本没有用到登录，那创建登录框这个操作就是无用的，所以改进为当用户点击登录按钮时才开始创建登录框，如下：
![](https://img2018.cnblogs.com/blog/1852401/202001/1852401-20200113114128801-1289386007.png)

<!--
```html
<body>
    <button id="loginbtn">登录</button>
    <script>
        const btn = document.querySelector("#loginbtn");
        const loginDiv = function () {
            const div = document.createElement("div");
            div.innerHTML = "登录浮窗";
            div.style.display = "none";
            document.body.appendChild(div);
            return div;
        }
        btn.onclick = function () {
            const loginDivElem = loginDiv();
            loginDivElem.style.display = "block";
        }
    </script>
</body>
```
-->

现在达到了*惰性*的目的，但失去了单例的效果。每次点击登录都会创建一个登录框，所以还要加个变量控制这个框唯一显示：
![](https://img2018.cnblogs.com/blog/1852401/202001/1852401-20200113115805075-985965257.png)

<!--
```javascript
      <button id="loginbtn">登录</button>
    <script>
        const btn = document.querySelector("#loginbtn");
        const createLoginDiv = (function () {
            const div = null;
            return function(){
                if(!div){
                    document.createElement("div");
                    div.innerHTML = "登录浮窗";
                    div.style.display = "none";
                    document.body.appendChild(div);
                }
                return div;
            }
        })();
        btn.onclick = function () {
            const loginDivElem = createLoginDiv();
            loginDivElem.style.display = "block";
        }
    </script>
```
-->
这里使用了闭包，将控制变量放到闭包里，防止全局污染，当第一次点击时，创建并显示登录框，再次点击后，不再进行创建直接显示。
这样就实现了这个登录框的惰性单例模式，但是他存在以下问题：
1. 仍然违反了单一职责原则，管理单例和创建对象的代码放在了同一函数内部。
2. 如果以后我想创建一个单例的表格，一个单例的iFrame，还要每次都这样重复写一遍，如法炮制

所以有没有一种通用的方法抽象这种模式呢？
# 通用的单例模式
先不考虑创建的东西是什么，我们先把管理单例的代码抽象出来，创建对象的方法当做此单例函数的参数传入：
![](https://img2018.cnblogs.com/blog/1852401/202001/1852401-20200113151303475-417709371.png)

<!--
```javascript
const getSingle = function(fn){
    let result = null;
    return function(){
        return result || (result = fn.apply(this, arguments))
    }
}
const createLoginDiv = function(){
    const div = document.createElement("div");
    div.innerHTML = "登录浮窗";
    div.style.display = "none";
    document.body.appendChild(div);
    return div;
}

const createSingleLoginDiv = getSingle(createLoginDiv);

loginBtn.onclick = function(){
    const div = createSingleLoginDiv();
    div.style.display = "block";
}
```
-->
我们创建了一个通用的单例函数，只专注与单例相关的代码，接受一个fn参数，将fn的执行结果存到result中，result由于在函数闭包里，所以不会被销毁。将来使用时result如果是已赋值的，那么直接返回这个单例值，这样如果需要创建一个单例的iFrame,只需要写一个代码逻辑与createLoginDiv差不多的createiFrame，放到getSingle里返回即可使用。
通过结合前一篇[[JavaScript设计模式] 什么是单例模式](https://www.cnblogs.com/hansonfang/p/12177396.html)的学习，我们了解了单例模式，尤其以惰性单例模式这种概念，在需要的时候才去创建对象，而且是唯一的对象。还有单一职责原则，创建对象和管理单例被放在两个不同的方法中，组合起来以达到我们想要的目的。