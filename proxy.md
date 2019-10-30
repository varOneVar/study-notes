## proxy对象

> ```
> let p = new Proxy(target, handler);
> // target: 用Proxy包装的目标对象（可以是任何类型的对象，包括原生数组，函数，甚至另一个代理）
> // handler: 一个对象，其属性是当执行一个操作时定义代理的行为的函数
> ```



#### 能劫持的13个api

~~~js

handler.getPrototypeOf()

// 在读取代理对象的原型时触发该操作，比如在执行 Object.getPrototypeOf(proxy) 时。

handler.setPrototypeOf()

// 在设置代理对象的原型时触发该操作，比如在执行 Object.setPrototypeOf(proxy, null) 时。

handler.isExtensible()

// 在判断一个代理对象是否是可扩展时触发该操作，比如在执行 Object.isExtensible(proxy) 时。

handler.preventExtensions()

// 在让一个代理对象不可扩展时触发该操作，比如在执行 Object.preventExtensions(proxy) 时。

handler.getOwnPropertyDescriptor()

// 在获取代理对象某个属性的属性描述时触发该操作，比如在执行 Object.getOwnPropertyDescriptor(proxy, "foo") 时。

handler.defineProperty()

// 在定义代理对象某个属性时的属性描述时触发该操作，比如在执行 Object.defineProperty(proxy, "foo", {}) 时。

handler.has()

// 在判断代理对象是否拥有某个属性时触发该操作，比如在执行 "foo" in proxy 时。

handler.get()

// 在读取代理对象的某个属性时触发该操作，比如在执行 proxy.foo 时。

handler.set()

// 在给代理对象的某个属性赋值时触发该操作，比如在执行 proxy.foo = 1 时。

handler.deleteProperty()

// 在删除代理对象的某个属性时触发该操作，比如在执行 delete proxy.foo 时。

handler.ownKeys()

// 在获取代理对象的所有属性键时触发该操作，比如在执行 Object.getOwnPropertyNames(proxy) 时。

handler.apply()

// 在调用一个目标对象为函数的代理对象时触发该操作，比如在执行 proxy() 时。

handler.construct()

// 在给一个目标对象为构造函数的代理对象构造实例时触发该操作，比如在执行new p

~~~



#### 观察者模式

~~~js
//初始化观察者队列
const arr = new Set();

//将监听函数加入队列
const obe = fun => {
  arr.add(fun);
}

//初始化Proxy对象，设置拦截操作
const observable =  obj => new Proxy(obj, {set});

function set(target, key, value, receiver){
     //内部调用对应的 Reflect 方法
     const result = Reflect.set(target, key, value, receiver);
     //额外执行观察者队列
     arr.forEach( item => item() );
     return Reflect.set(target, key, value, receiver);
}

const target = {
   name:"小明",
   age:15
}


const per = observable(target);

function print(){
   console.log( per.name);
}


obe(print);

per.name = "小红"
 //结果   小红,15
~~~



