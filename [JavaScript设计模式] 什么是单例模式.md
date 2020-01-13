# 概念
> 保证一个类仅有一个实例，并提供一个全局访问点
## 为什么要用单例模式
想象一下某些web应用，当点击登录按钮时，会弹出一个登录框，无论你点击多少次这个登录按钮，登录框都只会出现一个，不会出现多个登录框。同时不会频繁的进行删除和添加，而是同一个登录框进行隐藏和显示，因为删除和添加十分耗费性能，所以单例可以达到最大化的效能利用。登录框这个例子就是单例模式最典型的应用，符合业务的需求，又能够提高性能
# 单例模式的实现
## 简单的单例模式
一个简单的单例模式，无非就是用一个变量指示要创建的实例是否已经创建过了，如果已经创建过了，则在下一次使用实例时，直接返回复用，如果没有创建过，则创建并保存到变量中。
![](https://img2018.cnblogs.com/blog/1852401/202001/1852401-20200110184150647-532095394.png)

<!--
```javascript
const SingleTon = function(name){
    this.name = name;
}
SingleTon.prototype.getName = function (){
    return this.name;
}
SingleTon.instance = null;
SingleTon.getInstance = function(name){
    if(!this.instance){
        this.instance = new SingleTon(name);
    }
    return this.instance;
}
const aaa = SingleTon.getInstance("aaa");
const bbb = SingleTon.getInstance("bbb");
aaa.getName() === bbb.getName();    // true
```
-->

但是这时候的这个单例类是“不透明”的，因为我们通常习惯使用`new XXX()`的方式来实例化一个类，而不是通过使用者不知道的`XXX.getInstance()`的方式来获取单例对象。
所以我们应该使用**透明的单例类**
## 透明的单例类
先介绍下IIFE
> IIFE（ 立即调用函数表达式）是一个在定义时就会立即执行的  JavaScript 函数 --- MDN

eg.
![](https://img2018.cnblogs.com/blog/1852401/202001/1852401-20200112174833694-1583208094.png)
<!--
```javascript
    const foo = (function(){
        return "I am foo"
    })()
    console.log(foo); // "I am foo"
```
-->
之所以使用IIFE，就是限制JS的变量作用域，在ES5中，没有块级作用域，只有函数作用域一种，所以提供一种模拟块级作用域的方法就是IIFE。我们的透明单例模式就是使用IIFE来模拟。
![](https://img2018.cnblogs.com/blog/1852401/202001/1852401-20200112175850592-1937704270.png)

<!--
```javascript
  const CreateDiv = (function(){
        let instance = null;
        const CreateDiv = function(html){
            if(instance){
                return instance;
            }
            this.html = html;
            instance = this;
            return instance;
        }

        CreateDiv.prototype.init = function(){
            const div = document.createElement("div");
            div.innerHTML = this.html;
            document.body.appendChild(div);
        }
        return CreateDiv;
    })();
    
    const aaa = new CreateDiv("aaa");
    const bbb = new CreateDiv("bbb");
    aaa.init(); 
    bbb.init();
```
-->

使用IIFE来避免变量污染，把在IIFE中创建的类return出去，供外部通过`new XXX()`的形式调用。缺点是增加了程序的复杂度，不利于阅读。
在构造函数CreateDiv中：

<!--
```javascript
 const CreateDiv = function(html){
            if(instance){
                return instance;
            }
            this.html = html;
            instance = this;
            return instance;
   }
```
-->

实际上做了两件事，
1. 保证类只有一个实例
2. 构造实例

这样就不符合单一职责原则，这个构造函数就变得不纯粹了。如果哪天我不想使用单例了，我要在页面创建普通的实例，创建多个不一样的div，还需要去修改CreateDiv构造函数，去掉控制单一实例的那一段，但页面中使用此单例的代码可能就用不了，故会带来不必要的麻烦。所以我们应该使用单例代理。

## 使用代理的单例类
上面的问题我们可以用单例代理来解决，通过引入一个代理类，代理普通CreateDiv类，使之变为单例类。
![](https://img2018.cnblogs.com/blog/1852401/202001/1852401-20200112182950991-1783649740.png)
<!--
```javascript
const CreateDiv = function (html) {
    this.html = html;
}
CreateDiv.prototype.init = function () {
    const div = document.createElement("div");
    div.innerHTML = this.html;
    document.body.appendChild(div);
}
const ProxyCreateDiv = (function () {
    let instance = null;
    return function (html) {
        if (!instance) {
            instance = new CreateDiv(html);
        }
        return instance;
    }
})();
const aaa = new ProxyCreateDiv("aaa");
const bbb = new ProxyCreateDiv("bbb");
aaa.init(); // aaa
bbb.init(); // aaa
```
-->

我们把负责单例类管理的逻辑放到了单独的ProxyCreateDiv类中，CreateDiv类还是一个普通的创建实例的类，这样保证了单一职责原则，组合起来就能达到单一职责原则。

以上就是单例类的创建模式，还有一类单例模式的应用，就是文章开头说的登录框，点击登录按钮，登录框会弹出，而无论点击多少次，登录框都只会弹出一个，这个登录框就是单例的，稍后我会新写一篇文章分享这种单例模式的应用，敬请关注。
