> 学好typescript，走遍天下都不怕                                  -----卡夫卡 

###  基础写法

~~~typescript
/* ==== 基本类型定义 ==== */

let isTure: boolean = false; // 布尔
let num: number = 6; // 数值
let str: string = '字符串'
let u: undefined = undefined;
let n: null = null;
let obj: {x: number, y:number} = {x:1,y:2} // 对象， 如果直接obj:object是无法obj.x再赋值的，因为没有定义类型

# 数组
let list: string[] = ['1','b', 'c']; 
let list: Array<number> = [1,2,3];
let list: Array<number|string> = [1,'2',3]; // 多类型
/* ==== ts 特殊类型 ==== */

# 元组Tuple: 特俗的数组，定义数组个数和对应类型
// 元组可以使用数据的push添加元素，但是添加的元素越界操作会报错: tuple[push下标]，不建议使用
let x: [string, number, boolean]; // 定义
x = ['hello', 1. false]; // 赋值，只能按照定义类型顺序
x[8] = 'asd' // 数组越界赋值会使用(string|number|boolean)联合类型替代，不建议越界赋值

# 枚举enum：编译为对象
// 分为数字枚举，字符串枚举，异构枚举(混合)
// 数字枚举编译后是反向映射生成的：let color = {}; color[color['red']=1] = 'red';输出color结果是{1:'red',red:1};
// 字符串枚举就是简单赋值，常规对象
// 异构枚举容易混淆，不建议使用

// 枚举成员只读
// 枚举成员分为常量枚举成员和计算枚举成员，常量枚举成员包括
enum Char {
    // const:编译期就计算结果
    a,
    b=Char.a,
    c=1+2,
    // computed:运行时环境计算结果
    d=Math.random(),
    e='123'.length,
}
// 常量枚举: const声明，编译后没有任何代码
const enum Month {
    Jan, Feb, Mar
}
// 不需要对象，只要对象值的时候，就可以使用常量枚举，减少编译代码
const a = [Month.Jan, Month.Feb, Month.Mar]

// 枚举类型
enum E { a, b } // 原始
enum F { a = 0, b = 1 } // 数字
enum G { a = 'apple', b = 'banana'} // 字符串

let e: E = 3; // 可以把任何number类型赋值给枚举类型
let f: F = 3 // 取值可以超出枚举成员的定义
// err： e === f ; 不同类型的枚举之间不能比较

let e1: E.a
let e2: E.b
let e3: E.a
// err: e1 === e2 ; 同枚举类型下的不同枚举成员之间也不是不能比较
// success e1 === e3 ; // 同枚举类型同枚举成员定义就可以比较

let g1: G = G.a || G.b // 字符串类型的赋值只能是定义的枚举成员
let g2: G.a = G.a // 指定了枚举成员，赋值就只能是指定的这个成员

enum Color { Red=1, Green, Blue }; // 默认从0开始编号，可以修改，后面顺延
let c: Color = Color.Red; // c:1,
let colorName: string = Color[2] // colorName:'Green',通过编号拿到枚举的名字
// 实例
const enum MediaTypes {
  JSON = "application/json"
}

fetch("https://swapi.co/api/people/1/", {
  headers: {
      Accept: MediaTypes.JSON
  }
})
.then((res) => res.json())

# any与void
// any就是任意类型，不赘述
// 函数没有返回值就用void变量一般没使用void的，使用void只能赋值undefined和null
function warnUser(): void {
    console.log("This is my warning message");
}
// never，一般都是抛错，以及死循环函数使用，除了never本身，没有其他可以赋值给never类型
function error(message: string): never {
    throw new Error(message);
}
function infiniteLoop(): never {
    while (true) {
    }
}

# 函数的定义
// 1.直接编写，返回类型会自动推导，没必要定义返回类型
function add(x:number, y: number) {
    return x+y;
}
// 2. 未实现具体函数
let add:(x:number, y: number) => number
// 3. 未实现具体函数
type add = (x:number, y: number) => number;
// 4. 未实现具体函数
interface add {
    (x:number,y:number):number
}
# 函数重载
// 先定义不同类型的同名函数申明作为类型列表
// 然后使用最宽泛的类型实现函数，ts会从上到下去匹配合适的类型定义；
function add(...rest:number[]): number;
function add(...rest:string[]): string;
function add(...rest:any[]):any {
    let first = rest[0];
    if (typeof first === 'string') {
        return rest.join('-')
    }
    if (typeof first === 'number') {
        return rest.reduce((pre,cur) => pre + cur)
    }
}
~~~

#### 类型断言

~~~typescript
// 值 as 类型 或者 <类型>值,推荐as写法，因为jsx语法只能as断言
// 类型断言必须联合类型中的一种，类型断言只是做类型选择，而不是做类型转换。
let a: string | number = 123;
let b: number = a; // 这里b会报错
let b: number = a as number; // 做类型断言就告诉编译到底是什么，避免报错

// 括号包裹使用api
let someValue: any = "this is a string";
let strLength: number = (someValue as string).length;

~~~

#### interface 与 type

~~~~typescript
/* =============== interface ============= */
// 相当于mongoose里schema的概念
interface labelledValue {
    label: string;
    width?:number; // 可选参数
    [x: string]: any; // 任意数量，任意类型
}
// 传入参数必须有label且类型为string, 如果
function printLable(labelledObj:labelledValue) {
    console.log(labelledObj);
}
// 如果没有[x: string]: any;，传入数据有多余字段时，直接传字面量对象会报错
// 有三种方式避开错误: 
// 1. 接口定义中[x: string]: any;
// 2. 将字面量赋值给变量，将变量代替字面量传入
// 3. 类型断言： printLable({ size: 10, label: '大小', kkk: '23123' } as labelledValue)
printLable({ size: 10, label: '大小', kkk: '23123' })

// 定义字符串数组
interface StringArray {
    [index: number]: string;
}
let chars:StringArray = ['sadsad', 'dd']

interface Names {
    [x:string] : string; // 用任意字符串去索引Names,得到的结果是string
    // u: number;  错误，上面已定义死了
    [z: number]: string; // 可以
    // [z: number]: number; 不行，除非上面的定义的是[x:string] : any; 只有是上面的子级类型才行
}

// readonly，用于对象定义，定义一般变量用const就好了
interface Point {
    readonly x: number;
    readonly y: number;
}
let p1: Point = { x: 10, y: 20 };
p1.x = 5; // error!

# ReadonlyArray<number> 可以定义只读数组
let a: number[] = [1, 2, 3, 4];
let ro: ReadonlyArray<number> = a;
ro[0] = 12; // error!
ro.push(5); // error!
ro.length = 100; // error!
a = ro; // error! 整个ReadonlyArray赋值到一个普通数组也不行
a = ro as number[]; // ok，通过断言重写才可以

# 函数类型接口
interface SearchFunc {
  (source: string, subString: string): boolean;
}
// 函数参数名不用与接口定义名相同，只要类型和返回类型一致就可以
// 后面函数参数和返回值可以不写类型，类型推导会根据接口来做类型校验
let mySearch: SearchFunc = function(src: string, sub: string): boolean {
  let result = src.search(sub);
  return result > -1;
}

# 类类型接口
// 类实现了一个接口时，只对其实例部分进行类型检查
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date);
}
class Clock implements ClockInterface {
    currentTime: Date;
    setTime(d: Date) {
        this.currentTime = d;
    }
    constructor(h: number, m: number) { }
}

# 继承接口
interface Shape {
    color: string;
}
interface PenStroke {
    penWidth: number;
}
interface Square extends Shape, PenStroke {
    sideLength: number;
}
let square = <Square>{};
square.color = "blue";
square.sideLength = 10;
square.penWidth = 5.0;

# 接口继承基类
class Control {
    private state: any;
}
interface SelectableControl extends Control {
    select(): void;
}
class Button extends Control implements SelectableControl {
    select() { }
}
class TextBox extends Control {
    select() { }
}
// 错误：“Image”类型缺少“state”属性。
class Image implements SelectableControl {
    select() { }
}

/* =============== interface与type的异同 ============= */
# 都可以描述一个对象或函数
# 都允许继承，可以互相继承
// type的继承写法
type Name = { 
  name: string; 
}
type User = Name & { age: number  };

# type可以，interface不行
type Name = string // 基本类型别名
type Text = string | { text: string }; 
// 联合类型
interface Dog {
    wong();
}
interface Cat {
    miao();
}
type Pet = Dog | Cat
// 具体定义数组每个位置的类型
type PetList = [Dog, Pet]
// 利用typeof获取变量类型
let div = document.createElement('div');
type B = typeof div

# interface可以，type不行
interface User {
  name: string
  age: number
}

interface User {
  sex: string
}
合并了声明
/*
User 接口为 {
  name: string
  age: number
  sex: string 
}
*/
# 能用 interface 实现，就用 interface 
~~~~



#### 泛型

~~~typescript
/* ===============定义============= */
// 函数声明式
function generics1<T>(arg: T[]): T[] { // 或者T[]替换成Array<T>
    return arg;
}

// 函数表达式
let generics2: <U>(arg: U) => U = function (arg) { // T可以是任意字符
    return arg;
}

// 对象字面量
let generics3: {<T>(arg: T): T} = identity;

// 泛型接口
interface GenericIdentityFn<T> {
    (arg: T): T;
}
function identity<T>(arg: T): T {
    return arg;
}
let myIdentity: GenericIdentityFn<number> = identity;

// 泛型类型
type Predicate<T> = (v: T) => boolean;
class FilteredList<T> {
    filter: Predicate<T>;
    data: T[];
    constructor(filter: Predicate<T>) { ... }
    add(value: T) { ... }
    get all(): T[] { ... }
}

// 泛型类
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}
let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };

/* =============== 使用 ============= */

let output = generics2<string>("myString"); // 明确类型
let output = generics2("myString"); // 类型推论 -- 即编译器会根据传入的参数自动确定T的类型

/* =============== 泛型约束 ============= */
interface IAnimal {
    run(): void;
}
// 此时的extends表示约束关系，非继承；表示T实现了IAnimal接口里的类型
function run<T extends IAnimal>(animal: T): T {
    animal.run();   // it's ok
    return animal;
}


~~~

#### i

~~~typescript

~~~

