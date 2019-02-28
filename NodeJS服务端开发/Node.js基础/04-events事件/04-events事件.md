# Node.js 事件

JavaScript客户端编程中，事件是一个很重要的概念，因为客户端中响应用户操作是最重要的事。

Node.js中同样具有事件机制，这些功能在模块`events`中，这里简要介绍事件发射器（EventEmitter）的使用，详细请参考官方文档。

## 创建事件发射器和触发事件

下面是使用EventEmitter的一个例子：

```javascript
//创建一个事件对象
var events = require("events");
//绑定EventEmitter
var myEmitter = new events.EventEmitter();
myEmitter.on("someEvent", function(message){
    console.log(message);
});

//触发事件
myEmitter.emit("someEvent", "hello, world!");
```

代码比较简单，就不多做解释了。

## 继承EventEmitter

下面代码中，实现了继承`EventEmitter`的类，并为其实例化的对象绑定了事件。我们可以对比一下`Java/RxJava响应式编程/01-异步回调函数`中编写的代码，显然JavaScript代码在这里表现的更加灵活，甚至可读性更强，Java则像是绕了一圈才达到我们的目的，Java中回调函数的写法让很多初级程序员费解。

```javascript
var events = require("events");

class Student extends events.EventEmitter
{
    constructor(name) {
        super();
        this.name = name;
    }
    speak () {
        console.log("Hello, I am " + this.name);
    }
}

var tom = new Student("Tom");
var cat = new Student("Cat");

tom.on("speak", function() {
    tom.speak();
});

cat.on("speak", function() {
    cat.speak();
});

tom.emit("speak");
cat.emit("speak");
```
