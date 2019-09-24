# typescript cookbook

## 1.类型

### 数组

```typescript
// 类型数组表示
const list:number[] = [1,2,3]
```

```typescript
// 范型数组表示
const list:Array<number> = [1,2,3]
```



### 元组tuple

多类型数组

> 某些函数的参数能用上
>
> 提供友好的api方便解构赋值

```typescript
const msg:[string,number] = ["ray", 28]
```



### 枚举

如果值为number类型（默认），则支持反查（ws会报警）

正常情况用默认或给初值即可



### Any

> 动态内容：仅在用户输入或者第三方代码库使用

跳过类型检查器检查



### Void

- 没有返回的函数
- undefined和null



### Null和Undefined

--strictNullChecks

```typescript
// 联合类型
type input = string | null | undefined
```



### Never

永不存在的值的类型

- 函数
  - 必定抛出异常
  - 死循环
- 变量
  - 是任何类型的子类型，也可以赋值给任何类型
  - *没有*类型是`never`的子类型或可以赋值给`never`类型



### 类型断言

类似类型转换，但只在编译阶段生效（编译完的js没有意义）

```typescript
// 尖括号
const input:any = 'input'
const len:number = (<string>input).length
// as
const len2:number = (input as string).length
```



---



## 2.变量声明

### 解构

#### 属性重命名

```typescript
// 太繁琐了吧不推荐这种写法
const {age:newAge,name}:{age:number,name:string} = info
```

#### 默认值

```typescript
// 有点难读
function foo (obj:{a:string,b:?number}){
  let {a,b=100} = obj
}
// 使用type
type FooParams = {a:string,b?number}
function foo(obj: FooParams){}
```

> ！小心使用解构作为函数参数，嵌套起来难以理解且容易出错,最好就是别用
>
> ！TypeScript编译器不允许展开泛型函数上的类型参数



---



## 3.接口

**这个是重点**

使用struct type进行契约编程

### 可选属性

```typescript
interface Config{
	color?: string
}
```

### 只读属性

```typescript
interface Atom{
  readonly type: string
}
```

#### readonly数组

```typescript
const roList:ReadonlyArray<number> = [1, 2, 3]
// 类型和Array不兼容，需要类型断言
const a: Array<number> = roList as Array<number>
```

### 额外的属性检查

 对象字面量会被特殊对待而且会经过 *额外属性检查*，当将它们赋值给变量或作为参数传递的时候。 如果一个对象字面量存在任何“目标类型”不包含的属性时，你会得到一个错误。

> 可以通过类型断言跳过检查，但不推荐

#### 字符串索引签名

> 严格的设计用不上这个，最好也不要用，除非受第三方影响

```typescript
interface Config{
  [propName: string]: any
}
```

### 函数类型

显示声明函数的输入输出类型

```typescript
interface RandomFunc{
  (range: number): number;
}
const myRandom: RandomFunc = function(range){
  return _.random(range)
}
const myRandom2: RandomFunc = function(range:number):number{
  return _.random(range)
}
```

> 这里不再重复声明函数的类型，因为在接口已经声明了，所以函数类型的声明尽量靠近函数定义
>
> 如果定义和声明较远则声明类型
>
> 参数名不作校验,但尽量一致

### 可索引的类型

支持两种索引签名：字符串和数字

可以同时使用两种类型的索引，但是数字索引的返回值必须是字符串索引返回值类型的子类型(编译器会把数字视为字符串)

能够很好的描述`dictionary`模式

```typescript
interface Dictionary{
	[index: number]: number,
  name: string //error (should be number)
}
```

能创建只读类型数组

```typescript
interface ReadOnlyStringArray{
  readonly[index:nuber]: string
}
```

### 类类型

#### 实现接口

`implements`

接口仅描述类的公共部分

#### 类静态部分与实例部分的区别

##### 构造器签名

```typescript
interface User {
  name: string
}
interface Ctor {
  new (name: string): User
}
```

构造器签名接口用于类静态部分

> 一个类可能需要实现两个接口，分别实现类静态部分和类实例部分, 静态部分不声明静态属性，只包含构造器签名

### 继承接口

> 可视为成员复制

```typescript
// 类型断言+接口 延迟初始化属性（不推荐使用）
interface Config{
    use: boolean
}
const config = <Config>{}; // 接口的属性可稍后添加
// const config:Config = {} 这种方式会直接报错
config.use = true;
```

> 支持多继承，但最好还是别用

### 混合类型

对象可以同时做为函数和对象使用，并带有额外的属性

> 除了少数对外api，不推荐使用

### 接口继承类

继承类的成员但不包括其实现，包括private和protected成员

当你创建了一个接口继承了一个拥有私有或受保护的成员的类时，这个接口类型只能被这个类或其子类所实现

> 没想到应用场景



---



## 4.类

在构造函数里访问 `this`的属性之前，我们 *一定*要调用 `super()`

#### 公共，私有与受保护的修饰符

默认标记为public

> 明确的声明public

#### 理解private

TypeScript使用的是结构性类型系统。当我们比较两种不同的类型时，并不在乎它们从何处而来，如果所有成员的类型都是兼容的，我们就认为它们的类型是兼容的

但使用private或protected时，此规则不再适用

#### 理解 protected

`protected`成员在派生类中仍然可以访问

##### protected 构造函数

意味着这个类不能在包含它的类外被实例化，但是能被继承

### readonly修饰符

必须在声明时或构造函数里被初始化

> 类似接口readonly

#### 参数属性

把声明和赋值合并至一处

> 感觉不如属性声明直观，除非参数特别少

```typescript
class Person {
    constructor (readonly name: string) {}
}
class Person {
    constructor (private readonly name: string) {}
}
```

### 存取器

只带有 `get`不带有 `set`的存取器自动被推断为 `readonly`。 这在从代码生成 `.d.ts`文件时是有帮助的，因为利用这个属性的用户会看到不允许够改变它的值。

### 抽象类

- 不会直接被实例化

- 不同于接口，抽象类可以包含成员的实现细节

- `abstract`关键字是用于定义抽象类和在抽象类内部定义抽象方法

> 注意abstract readonly public/proteced/private的混合使用

### 高级技巧

#### 构造函数

##### 获取构造函数类型

```typescript
class Book{}
const BookCtor: typeof Book = Book
```

#### 把类当做接口使用

类的实例类型和一个构造函数

> 大部分时间用不上吧



---



## 5.函数

### 函数类型

#### 函数定义类型

- 声明每个参数类型
- 声明返回值类型（通常利用类型推断省略）

```typescript
// 普通声明
const add = (x:number,y:number):number => x+y
const add2 = (x:number,y:number) => x+y
```

#### 完整函数类型

```typescript
const add: (base:number,increase:number)=>number = (x:number,y:number):number => x+y
```

> 有点繁琐
>
> 完整函数类型必须声明返回值类型
>
> 不包含闭包的变量

##### 上下文归类

```typescript
const add: (base:number,increase:number)=>number = (x,y) => x+y
```

>普通声明和上下文归类二选一即可

### 可选参数和默认参数

传递给一个函数的参数个数必须与函数期望的参数个数一致

```typescript
function printName(name?:string){
  const content = name ?? 'ray'
  console.log(content)
}
function printName(name:string='ray'){
  console.log(content)
}
//typeof printName === (name?:string) => void
```

在所有必须参数后面的带默认初始化的参数都是可选的

可选参数与末尾的默认参数共享参数类型

与普通可选参数不同的是，带默认值的参数不需要放在必须参数的后面

### 剩余参数

> 少数可能用tuple类型的地方，但用数组更好
>
> 除非编写一下函数式的库函数，最好别用



### `this`

`this`的值在函数被调用的时候才会指定。 这是个既强大又灵活的特点

`--noImplicitThis`:当`this`表达式的值为`any`类型的时候，生成一个错误

#### `this`参数

```typescript
interface GameConsole{
  games: Game[]
  play: (this:GameConsole) => Game
}
```

#### `this`参数在回调函数里

1. 库函数的作者要指定 `this`的类型
2. 调用者需要声明匹配的this类型或者使用箭头函数(箭头函数不会捕获`this`)



### 重载

在定义重载的时候，一定要把最精确的定义放在最前面

```typescript
function foo(a: string): number;
function foo(a: number): string;
//any并不是重载列表的一部分
function foo(a: any): any {
    if (typeof a === 'number') {
        return 0
    } else {
        return 1
    }
}
```

> 重载的实现动态部分需要声明为any
>
> 重载的检测比可选参数更加严格，因此推荐需要使用可选参数时使用重载声明（如果可以），并在实现时使用可选参数



---



## 6.范型

泛型常用参数名：T，U，K，V

> 泛型就是：给类型传参

### 目的

- 创建一致的定义良好的API
- 支持未来的数据类型
- 特别适用于函数式编程

```typescript
function ensureArray<T>(input: Array<T>): [T];
function ensureArray<T>(input: T): [T];
function ensureArray(input: any) {
    if (!Array.isArray(input)) {
        return [input]
    }
    return input
}
// 使用尖括号声明范型的具体类型
const a: [number] = ensureArray<number>(123);
// 使用类型推断
const c: [number] = ensureArray([1, 2, 3]);
```

> 不同于any, 范型能够声明参数与返回值的类型相关性，便于编译器检测和类型推断

> 除非迫不得已，推荐使用类型推断，保持代码精简和高可读性

### 使用泛型变量

类型变量代表的是任意类型

### 范型类型

```typescript
// 范型函数类型
const toArray: <T>(input:T) => Array<T> = function(input){
  return [input]
}
// 使用带有调用签名的对象字面量来定义泛型函数
const toArray2:{<U>(userInput:U)=>Array<U>} = toArray
// 范型接口
interface ToArrayFn {
    <T>(input: T): Array<T>
}
const toArray3: ToArrayFn = userToArray;
// 把泛型参数当作整个接口的一个参数
interface ToArrayFn2<T> {
    (input: T): Array<T>
}
const toArray4: ToArrayFn2<string> = toArray3;
```

> 把泛型参数当作整个接口的一个参数,接口里的其它成员也能知道这个参数的类型,但调用时需要声明范型类型

### 泛型类

无法创建泛型枚举和泛型命名空间

与泛型接口差不多。 泛型类使用（ `<>`）括起泛型类型，跟在类名后面

泛型类指的是实例部分的类型，所以类的静态属性不能使用这个泛型类型

```typescript
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}
let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```

> 使用场景不明

### 泛型约束

> 使用接口和`extends`关键字来实现约束

```typescript
interface Lengthwise{
  length: number
}
function addLength<T>(list:T):T{
  list.length += 1 //不确定T是否包含length属性
  return list
}
function addLength<T extends Lengthwise>(list:T):T{
  list.length += 1
  return list
}
```

#### 在泛型约束中使用类型参数

`keyof` 操作符获取类型所属的key

```typescript
function getProperty<T,K extends keyof T>(obj: T, key: K) {
    return obj[key];
}
```

### 在泛型里使用类类型

> 工厂函数常用

```typescript
function factory<T>(Ctor:{new():T}):T{ //使用构造器签名
  return new Ctor()
}
// 使用范型接口
interface Ctor<T> {
    new(): T
}
function create<T>(c: Ctor<T>): T {
    return new c()
}
function createDate(c: Ctor<Date>): Date {
    return new c()
}
```

使用原型属性推断并约束构造函数与类实例的关系



---



## 7.枚举

> 分组的具名常量

支持数字的和基于字符串的枚举

#### 数字枚举

未声明的值将自动自增

使用计算值，则后面的枚举需要声明

```typescript
enum E {
  A = getValue(),
  B // error, need initializer
}
```

#### 字符串枚举

每个成员都必须用字符串字面量，或另外一个字符串枚举成员进行初始化

字符串枚举可以很好的序列化

> 推荐使用

#### 异构枚举

> 不推荐使用

#### 计算的和常量成员

枚举成员都带有一个值，它可以是 *常量*或 *计算出来的*

#### 联合枚举与枚举成员的类型

当所有枚举成员都拥有字面量枚举值时：

1. 枚举成员成为了类型, 该类型*只能*是枚举成员的值

```typescript
enum Atom{
  Image = 'Image'
}
interface ImageAtom{
  type:Atom.Image // type === Atom.Image
}
```

2. 枚举类型本身变成了每个枚举成员的 *联合*

无需再做额外的校验

#### 运行时的枚举

枚举是在运行时真正存在的对象

> 当成对象字面量

#### 反向映射

从枚举值到枚举名字

*不会*为字符串枚举成员生成反向映射

> 这功能几乎没用，使用字符串枚举，除非不需要持久化

#### `const`枚举

避免在额外生成的代码上的开销和额外的非直接的对枚举成员的访问

```typescript
const enum Enum {
    A = 1,
    B = A * 2
}
```

只能使用常量枚举表达式，编译阶段使用内联，编译完删除，不允许包含计算成员。

> 可以使用

#### 外部枚举

描述已经存在的枚举类型的形状

```typescript
declare enum Enum {
    A = 1,
    B, // 没有初始化方法时被当做需要经过计算的
    C = 2
}
```

> declare意味着这段代码不会编译到运行时，运行时的枚举需要外部引入



---



## 8. 类型推论

### 基础

推断发生在初始化变量和成员，设置默认参数值和决定函数返回值时

### 最佳通用类型

编译器会推断最合适的类型

- 尽可能精确
- 兼容所有候选类型
- 没有找到最佳通用类型的话，类型推断的结果为联合数组类型

> 不会自动推断出父类

### 上下文类型

上下文归类: 表达式类型被执行环境影响

- 执行环境不明时会被推断为any
- 手动声明类型可以覆盖上下文归类
- 应用函数的参数，赋值表达式的右边，类型断言，对象成员，数组字面量，返回值语句

上下文类型也会做为最佳通用类型的候选类型

> 会自动推断出父类



---



## 9.类型兼容性

### 介绍

类型兼容性基于结构子类型（structural subtyping）

结构类型是一种只使用其成员来描述类型的方式，与名义类型（nominal typing）形成对比

适用于匿名对象：函数表达式和对象字面量

```typescript
interface Named {
    name: string;
}
class Person {
    name: string;
}

let p: Named = new Person();
```

> 为兼容js而设计，最好不要依赖，使用名义类型进行设计



### 开始

结构化类型系统的基本规则:

- 如果`x`要兼容`y`，那么`y`至少具有与`x`相同的属性
- 只有目标类型的成员会被一一检查是否兼容
- 比较过程是递归进行的，检查每个成员及子成员
- y=x (y为目标，x为源)

### 比较两个函数

y=x

- 参数列表：`x`的每个参数必须能在`y`里找到对应类型的参数
  - 典型的比如forEach函数忽略参数问题
- 类型系统强制源函数的返回值类型必须是目标函数返回值类型的子类型

> 多的兼容少的

#### 函数参数双向协变

> 传入的函数参数更加准确时（如MouseEvent=>void） 需要用类型断言降级成声明的函数类型（Event=>void）

```typescript
function listenEvent(handler: (n: Event) => void) {
    /* ... */
}
// 参数兼容，使用时使用类型断言
listenEvent(e:Event => console((e as MouseEvent).x))
// 直接对函数使用类型断言
listenEvent(e:(MouseEvent => console(e.x)) as (n: Event) => void)
```

> 推荐直接对函数使用类型断言，最好配合接口使用

```typescript
interface EventHandler{
  (n:Event): void
}
function listenEvent(handler: EventHandler) {
    /* ... */
}
listenEvent(((e: MouseEvent) => console.log(e.x + "," + e.y)) as EventHandler)
```

#### 可选参数及剩余参数

比较函数兼容性的时候，可选参数与必须参数是可互换的。 当一个函数有剩余参数时，它被当做无限个可选参数

源类型上有额外的可选参数不是错误，目标类型的可选参数在源类型里没有对应的参数也不是错误。

> 如前所述，最好别用

#### 函数重载

需要覆盖所有函数签名

> 慎用

> 函数赋值容易引发不稳定，推荐使用接口声明



### 枚举

- 枚举类型与数字类型兼容

  - 数值不在枚举取值范围也不会引发编译错误

  ```typescript
  enum Color {
      red,
      blue,
      green
  }
  const c:Color = 3;
  ```

- 运行时类型与枚举类型兼容
  
  - 包含数字和字符串
- 不同枚举类型不兼容

> 配合枚举运行时使用，总的来说枚举需要考虑运行时，类型兼容也是基于运行时考虑



### 类

与对象字面量和接口

只比较实例部分：包括数据成员和子程序，且子程序类型需要类型兼容

> 尽量满足一致的抽象

#### 类的私有成员和受保护成员

private和protected影响兼容性，必须源于同一个类

> 满足一致的抽象



### 范型

范型类型未制定时默认为any

范型的类型兼容是基于带入范型类型的结果进行比较的

```typescript
// 数据成员不受T影响，因此始终兼容
interface Empty<T> {
}
let x: Empty<number>;
let y: Empty<string>;

x = y;  // OK, because y matches structure of x
```

```typescript
// 数据成员data受T影响，且number和string不兼容，因此不兼容
interface NotEmpty<T> {
    data: T;
}
let x: NotEmpty<number>;
let y: NotEmpty<string>;

x = y;  // Error, because x and y are not compatible
```

> 相当于范型预先执行了



### 高级主题

#### 子类型与赋值

赋值扩展了子类型兼容性，增加了一些规则，允许和`any`来回赋值，以及`enum`和对应数字值之间的来回赋值

类型兼容性是由赋值兼容性来控制的，即使在`implements`和`extends`语句也不例外



---



## 10.高级类型

### 交叉类型

将多个类型合并为一个类型

`&`操作符 `T & U`

适合mixin模式



### 联合类型

one of

竖线（ `|`）分隔符

```typescript
number | string | boolean
```

只能访问此联合类型的所有类型里共有的成员

### 类型保护与区分类型

访问联合类型各自的成员需要使用类型断言(每次都要) 

> 使用类型保护跳过这个限制

#### 用户自定义的类型保护

is操作符

`parameterName is Type`

```typescript
function isFish(pet: Fish|Bird): pet is Fish{ // pet 来自于参数列表 
  return (pet as Fish).swim !== undefined // 返回布尔值
}
if(isFish(pet)){ // pet 细化为Fish类型， 不在需要类型断言
  pet.swim()
}
```

#### `in`类型保护

```typescript
function move(pet: Fish | Bird) {
    if ("swim" in pet) {
        return pet.swim();
    }
    return pet.fly();
}
```

#### `typeof`类型保护

`typeof v === "typename"`和 `typeof v !== "typename"`， `"typename"`必须是 `"number"`， `"string"`， `"boolean"`或 `"symbol"`

#### `instanceof`类型保护

*instanceof类型保护*是通过构造函数来细化类型的一种方式

细化为:

1. 构造函数的prototype属性的类型,如果它的类型不为 `any`的话
2. 构造函数签名所返回的类型的联合

> ?不太明白



### 可以为null的类型

`--strictNullChecks`

#### 可选参数和可选属性

使用了 `--strictNullChecks`，可选参数会被自动地加上 `| undefined` ，可选属性亦如是

#### 类型保护和类型断言

##### null去除

1. `==`去除null

```typescript
function f(sn: string | null): string {
    if (sn == null) {
        return "default";
    }
    else {
        return sn;
    }
}
```

2. 短路运算符去除

```typescript
function f(sn: string | null): string {
    return sn || "default";
}
```

3. 类型断言手动去除

`!`后缀操作符

```typescript
function fixed(name: string | null): string {
  function postfix(epithet: string) {
    return name!.charAt(0) + '.  the ' + epithet; // !手动排除null
  }
  name = name || "Bob"; // name被修改了，此处编译器检测不到
  return postfix("great");
}
```

编译器无法去除嵌套函数的null，IIFE除外



### 类型别名

> 《代码大全》推荐写法, 屏蔽底层实现,提升代码抽象层次和可读性

`type` 关键字

```typescript
type Name = string
type Age = number
type User = {
    name: Name,
    age: Age
}
const user: User = {
    name: 'ray',
    age: 28
}
```

起别名不会新建一个类型,它创建了一个新 *名字* 来引用那个类型

类型别名支持泛型

```typescript
type Tree<T> = {
    value: T;
    left: Tree<T>;
    right: Tree<T>;
}
```

类型别名不能出现在声明右侧的任何地方

#### 接口 vs. 类型别名

1. 接口创建了一个新的名字，可以在其它任何地方使用
2. 类型别名不能被 `extends`和 `implements`(软件中的对象应该对于扩展是开放的，但是对于修改是封闭的)
3. 无法通过接口来描述一个类型并且需要使用联合类型或元组类型，这时通常会使用类型别名



### 字符串字面量类型

```typescript
type tagType = "img" | "input" | "div";
```

> 推荐配合联合类型使用

重载

```typescript
function createElement(tagName: "img"): HTMLImageElement;
function createElement(tagName: "input"): HTMLInputElement;
// ... more overloads ...
function createElement(tagName: string): Element {
    // ... code goes here ...
}

```

### 数字字面量类型

与字符串类似，较少使用

### 枚举成员类型

当每个枚举成员都是用字面量初始化的时候枚举成员是具有类型的

单例类型多数是指枚举成员类型和数字/字符串字面量类型



### 可辨识联合

合并单例类型，联合类型，类型保护和类型别名来创建一个叫做 *可辨识联合*的高级模式，它也称做 *标签联合*或 *代数数据类型*。 可辨识联合在函数式编程很有用处

1. 具有普通的单例类型属性— *可辨识的特征*。
2. 一个类型别名包含了那些类型的联合— *联合*。
3. 此属性上的类型保护。

```typescript
// 首先我们声明将要联合的接口
interface Square {
    kind: "square";
    size: number;
}
interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
}
interface Circle {
    kind: "circle";
    radius: number;
}
// kind属性称做 可辨识的特征或 标签。 其它的属性则特定于各个接口。

// 把它们联合到一起
type Shape = Square | Rectangle | Circle;
// 使用可辨识联合
function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.height * s.width;
        case "circle": return Math.PI * s.radius ** 2;
    }
}
```

#### 完整性检查

1. 启用 `--strictNullChecks`并且指定一个返回值类型

   ```typescript
   function area(s: Shape): number
   ```

2. 使用 `never`类型,编译器用它来进行完整性检查

   ```typescript
   function assertNever(x: never): never {
       throw new Error("Unexpected object: " + x);
   }
   function area(s: Shape) {
       switch (s.kind) {
           // cover case
           default: return assertNever(s); // error here if there are missing cases
       }
   }
   ```

   

### 多态的 `this`类型

多态的 `this`类型表示的是某个包含类或接口的 *子类型*。 这被称做 *F*-bounded多态性。

```typescript
class BasicCalculator {
    public add(operand: number): this {
        this.value += operand;
        return this;
    }
}
```

> 此处返回为this类型,而非BasicCalculator，因此继承BasicCalculator的类会返回相应的类，从而实现多态



### 索引类型

```typescript
function pluck<T, K extends keyof T>(o: T, names: K[]): T[K][] {
  return names.map(n => o[n]);
}
```

`keyof` : **索引类型查询操作符**

`keyof T`:`T`上已知的公共属性名的联合

```typescript
interface Person {
    name: string;
    age: number;
}
// keyof Person === 'name' | 'age' 注意Person为类型，而非实例
```



`T[K]`: **索引访问操作符**

确保类型变量 `K extends keyof T`

编译器会把`T[K]`映射为真实的类型

> 实现动态类型



#### 索引类型和字符串索引签名

`keyof`和 `T[K]`与字符串索引签名进行交互

```typescript
interface Map<T> {
    [key: string]: T;
}
let keys: keyof Map<number>; // string
let value: Map<number>['foo']; // number
```



### 映射类型

> 相当于可编程的类型系统，简化批量类型声明

```typescript
// 将一个已知的类型每个属性都变为可选的
type Partial<T> = {
    [P in keyof T]?: T[P];
}
// 上述类型不只一个成员，因此若要扩展需要使用交叉类型
type PartialWithNewMember<T> = {
  [P in keyof T]?: T[P];
} & { newMember: boolean }
```



```typescript
type Keys = 'option1' | 'option2';
type Flags = { [K in Keys]: boolean };
type Flags = {
    option1: boolean;
    option2: boolean;
}
```

1. 类型变量 `K`，它会依次绑定到每个属性。
2. 字符串字面量联合的 `Keys`，它包含了要迭代的属性名的集合。
3. 属性的结果类型。

实用的映射类型

```typescript
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
}
type Nullable<T> = { 
  [P in keyof T]: T[P] | null 
}
type Partial<T> = { 
  [P in keyof T]?: T[P] 
}

type Proxy<T> = {
    get(): T;
    set(value: T): void;
}
type Proxify<T> = {
    [P in keyof T]: Proxy<T[P]>;
}
function proxify<T>(o: T): Proxify<T> {
   // ... wrap proxies ...
}
let proxyProps = proxify(props);

type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
}
```

同态转换，保留T上所有存在的属性修饰器

```typescript
type Record<K extends string, T> = {
    [P in K]: T;
}
type ThreeStringProps = Record<'prop1' | 'prop2' | 'prop3', string>
```

非同态转换, 适合批量申明类型



#### 由映射类型进行推断

> 同态映射类型拥有完整的类型系统

```typescript
function unproxify<T>(t: Proxify<T>): T {
    let result = {} as T;
    for (const k in t) {
        result[k] = t[k].get();
    }
    return result;
}

let originalProps = unproxify(proxyProps);
```

##### 预定义的有条件类型

- `Exclude<T, U>` -- 从`T`中剔除可以赋值给`U`的类型。
- `Extract<T, U>` -- 提取`T`中可以赋值给`U`的类型。
- `NonNullable<T>` -- 从`T`中剔除`null`和`undefined`。
- `ReturnType<T>` -- 获取函数返回值类型。
- `InstanceType<T>` -- 获取构造函数类型的实例类型。

```typescript
Omit<T, K> === Pick<T, Exclude<keyof T, K>>
```

#### 条件类型

```typescript
T extends U ? X : Y // T如果是U的扩展，则类型为X
```

> 推断不出来则为联合类型（运行时布尔表达式不会执行）

```typescript
type TypeName<T> =
    T extends string ? "string" :
    T extends number ? "number" :
    T extends boolean ? "boolean" :
    T extends undefined ? "undefined" :
    T extends Function ? "function" :
    "object";

type T0 = TypeName<string>;  // "string"
type T1 = TypeName<"a">;  // "string"
type T2 = TypeName<true>;  // "boolean"
type T3 = TypeName<() => void>;  // "function"
type T4 = TypeName<string[]>;  // "object"
```

#### 分布式有条件类型

如果有条件类型里待检查的类型是`naked type parameter`，那么它也被称为“分布式有条件类型”。

```typescript
T = A | B | C
T extends U ? X : Y = (A extends U ? X : Y) | (B extends U ? X : Y) | (C extends U ? X : Y)
```

```typescript
type T10 = TypeName<string | (() => void)>;  // "string" | "function"
type T12 = TypeName<string | string[] | undefined>;  // "string" | "object" | "undefined"
type T11 = TypeName<string[] | number[]>;  // "object"
```

```typescript
type Diff<T, U> = T extends U ? never : T;  // Remove types from T that are assignable to U
type Filter<T, U> = T extends U ? T : never;  // Remove types from T that are not assignable to U
type NonNullable<T> = Diff<T, null | undefined>;  // Remove null and undefined from T
```

有条件类型与映射类型结合

```typescript
type FunctionPropertyNames<T> = { [K in keyof T]: T[K] extends Function ? K : never }[keyof T];
type FunctionProperties<T> = Pick<T, FunctionPropertyNames<T>>;

type NonFunctionPropertyNames<T> = { [K in keyof T]: T[K] extends Function ? never : K }[keyof T];
type NonFunctionProperties<T> = Pick<T, NonFunctionPropertyNames<T>>;
```

条件类型不允许递归引用

#### 有条件类型中的类型推断

`infer`表达式

它会引入一个待推断的类型变量。 这个推断的类型变量可以在有条件类型的true分支中被引用。 允许出现多个同类型变量的`infer`。

```typescript
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;
```

在这个条件语句 `T extends (...args: any[]) => infer R` 中，`infer R` 表示待推断的函数返回。

整句表示为：如果 `T` 能赋值给 `(...args: any[]) => infer R`，则结果是 `R`，否则返回为 `any`。

> infer常用于反解

在协变位置上，同一个类型变量的多个候选类型会被推断为联合类型

> 如果推断的类型为联合类型则返回联合类型

```typescript
type Foo<T> = T extends { a: infer U, b: infer U } ? U : never;
type T10 = Foo<{ a: string, b: string }>;  // string
type T11 = Foo<{ a: string, b: number }>;  // string | number
```

相似地，在逆变位置上(如函数参数)，同一个类型变量的多个候选类型会被推断为交叉类型：

```ts
type Bar<T> = T extends { a: (x: infer U) => void, b: (x: infer U) => void } ? U : never;
type T20 = Bar<{ a: (x: string) => void, b: (x: string) => void }>;  // string
type T21 = Bar<{ a: (x: string) => void, b: (x: number) => void }>;  // string & number 因为逆变，x会被推断为string和number的父类
```

##### 协变与逆变

- **协变**（covariant），如果它保持了[子类型序关系≦](https://link.zhihu.com/?target=https%3A//zh.wikipedia.org/wiki/%E5%AD%90%E5%9E%8B%E5%88%A5)。该序关系是：子类型≦基类型。
- **逆变**（contravariant），如果它逆转了子类型序关系。

绝大部分的语言是允许协变的，也就是上面说的子类型可以默认转换为父类型，逆变一般是不被允许的（除了函数的参数）。

> 函数之所以可以逆变是因为 
>
> 对于 Dog => Dog
>
> 参数声明为Animal的函数，输入Dog总是可以兼容的（Dog是Animal的子类，兼容所有Animal的接口）
>
> 返回值声明为Dog的函数，输出Husky总是可以兼容的 （Husky时Dog的子类，Husky总是兼容Dog）
>
> 因此对函数类型来说，输入为父类或输出为子类都是兼容的
>
> 对于Dog => Dog类型 ，Animal => Husky 类型的参数是兼容的
>
> 因此 Animal => Husky 是 Dog => Dog 的子类
>
> 返回值类型的子类型序关系是相同的，这个现象称为*协变*
>
> 很容易发现参数类型的子类型序关系是相反的，这个现象称为逆变
>
> ```
> A ≼ B 就意味着 (T → A) ≼ (T → B)
> A ≼ B 就意味着 (B → T) ≼ (A → T)
> ```



>在 `TypeScript` 中， [参数类型是双向协变的](https://github.com/Microsoft/TypeScript/wiki/FAQ#why-are-function-parameters-bivariant) ，也就是说既是协变又是逆变的，而这并不安全。但是现在你可以在 [`TypeScript 2.6`](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-6.html) 版本中通过 `--strictFunctionTypes` 或 `--strict` 标记来修复这个问题。



当推断具有多个调用签名（例如函数重载类型）的类型时，用*最后*的签名

无法在正常类型参数的约束子语句中使用`infer`声明

> 这一块还需要深入理解https://jkchao.github.io/typescript-book-chinese/tips/covarianceAndContravariance.html#一个有趣的问题



---



## 11. Symbols

运算符重载

> 强大而容易出错



---



## 12. 迭代器和生成器

> 生成器实现迭代器接口



## 13.模块

namespace: 内部模块

module: 外部模块

### 介绍

模块是自声明的；两个模块之间的关系是通过在文件级别上使用imports和exports建立的



### 导出

#### 导出声明

variable, function, class, type alias, interface

#### 导出语句

```typescript
export class Foo{} // 直接导出
export {Foo}
export {Foo as Bar} //导出重命名
```

#### 重新导出

```typescript
export {Foo as Baz} from 'Foo'
export * from 'Foo' 
```



### 导入

 #### 导入一个模块中的某个导出内容

```typescript
import {Foo} from 'Foo'
import {Foo as Bar} from 'Foo' //导入重命名
```

#### 将整个模块导入到一个变量，并通过它来访问模块的导出部分

```typescript
import * as FooModules from 'Foo'
FooModule.Foo
```

#### 具有副作用的导入模块

```typescript
import 'Foo'
```

> 不推荐，依赖副作用



### 默认导出

```typescript
//类和函数声明可以直接被标记为默认导出
export default class Foo{}
export default function bar(){}
//标记为默认导出的函数的名字是可以省略的。
export default function(){}
// default导出也可以是一个值
export default "123"
```

### export =` 和 `import = require()`

为了支持CommonJS和AMD的`exports`, TypeScript提供了`export =`语法。

`export =`语法定义一个模块的导出`对象`。 这里的`对象`一词指的是类，接口，命名空间，函数或枚举。

若使用`export =`导出一个模块，则必须使用TypeScript的特定语法`import module = require("module")`来导入此模块。

> 别用！



### 可选的模块加载和其它高级加载场景

使用import访问模块导出的类型

模块加载器会被动态调用（通过 `require`）

确保模块标识符只在类型注解部分使用，并且完全没有在表达式中使用时

```typescript
declare function require(moduleName: string): any;

import { ZipCodeValidator as Zip } from "./ZipCodeValidator";

if (needZipValidation) {
    let ZipCodeValidator: typeof Zip = require("./ZipCodeValidator"); // 使用typeof声明类型
    let validator = new ZipCodeValidator();
    if (validator.isAcceptable("...")) { /* ... */ }
}
```

> 一般情况下没必要复杂化



### 使用其它的JavaScript库

声明类库所暴露出的API

通常在 `.d.ts`文件里定义,当做C/C++ `.h`文件

##### 外部模块

```typescript
// user.d.ts
declare module "@/User"{
  export class User{
    name: string
  }
}
// User.js
class User{
  name
}
// index.ts
/// <reference path="user.d.ts"/> 这个能省略了
import { User } from "@/user"
class VipUser extends User{}
```

##### 外部模块简写

简写模块里所有导出的类型将是`any`

```typescript
declare module "@/user"
```

> 慎用

##### 模块声明通配符

```typescript
declare module "*!text" {
    const content: string;
    export default content;
}
// Some do it the other way around.
declare module "json!*" {
    const value: any;
    export default value;
}
import fileContent from "./xyz.txt!text";
import data from "json!http://example.com/data.json";
console.log(data, fileContent);
```

##### UMD模块

```typescript
export function isPrime(x: number): boolean;
export as namespace mathLib;
```

全局变量的形式使用，但只能在某个脚本（指不带有模块导入或导出的脚本文件）里



### 创建模块结构指导

> export default 被认为是有害的

##### 尽可能地在顶层导出

##### 使用命名空间导入模式当你要导出大量内容的时候

```typescript
import * as myLargeModule from "./MyLargeModule.ts";
```

##### 使用重新导出进行扩展

保持稳定的导出结构

```typescript
import { User, test } from "./User";
export class VipUser extends User {}
export {test} from './User'
```

##### 模块里不要使用命名空间

 在一个模块里，没有理由两个对象拥有同一个名字

> C#里My.Application.Customer.AddForm`和`My.Application.Order.AddForm通过命名空间区分

##### 危险信号

- 文件的顶层声明是`export namespace Foo { ... }` （删除`Foo`并把所有内容向上层移动一层）
- 多个文件的顶层具有同样的`export namespace Foo {` （不要以为这些会合并到一个`Foo`中！）



## 14.命名空间

> 总的来说没啥用

namespace

```typescript
namespace Scope {
  const a = 1
  export b = 2
}
console.log(Scope.b)
```

> 像不像iife返回的对象😌？

### 分离到多文件

#### 多文件中的命名空间

```typescript
/// <reference path="Scope.ts" />
namespace Scope{
  const c = 3
  export const d = 4
}
console.log(Scope.b)  
```

### 别名

```typescript
namespace Shapes {
    export namespace Polygons {
        export class Triangle { }
        export class Square { }
    }
}

import polygons = Shapes.Polygons;
let sq = new polygons.Square();
```

### 使用其它的JavaScript库

适合只提供少数的顶级对象的程序库，如d3

在.d.ts里声明，当作.h文件

#### 外部命名空间

不通过模块加载器加载

通过script标签加载

```typescript
declare namespace D3 {
    export interface Selectors {
        select: {
            (selector: string): Selection;
            (element: EventTarget): Selection;
        };
    }

    export interface Event {
        x: number;
        y: number;
    }

    export interface Base extends Selectors {
        event: Event;
    }
}

declare var d3: D3.Base;
```



## 15.命名空间和模块

### 使用命名空间

命名空间是位于全局命名空间下的一个普通的带有名字的JavaScript对象

> 最好别用



### 使用模块

依赖模块加载器

> 推荐使用



### 命名空间和模块的陷阱

#### 不要对模块使用`/// <reference>`

编译器查找顺序:

根据 `import`路径，编译器首先尝试去查找相应路径下的`.ts`，`.tsx`再或者`.d.ts`，如果这些文件都找不到，编译器会查找 *外部模块声明*（在.d.ts文件里声明，一般为node_modules）。

#### 不使用不必要的命名空间

```typescript
export namespace Scoep{} // don't do this
```



---



## 16.模块解析

1. 首先，编译器会尝试定位表示导入模块的文件
   -  Classic或Node策略
2. 如果上面的解析失败了并且模块名是非相对的（即 import "moduleA"），编译器会尝试定位一个外部模块声明

#### 相对 vs. 非相对模块导入

##### 相对引入

导入自己的模块

相对于文件位置

```typescript
import Entry from "./components/Entry";
import { DefaultHeaders } from "../constants/http";
import "/mod";
```

##### 非相对引入

导入外部模块

相对于`baseUrl`或通过路径映射来进行解析，还可以被解析成 外部模块声明

```typescript
import * as $ from "jQuery";
import { Component } from "@angular/core";
```



#### 模块解析策略

`--moduleResolution`标记来指定

`--module AMD | System | ES2015`时的默认值为[Classic](https://www.tslang.cn/docs/handbook/module-resolution.html#classic)，其它情况时则为[Node](https://www.tslang.cn/docs/handbook/module-resolution.html#node)

##### Classic

TypeScript默认的解析策略。 它存在的理由主要是为了向后兼容

查找 `moduleName.ts` `moduleName.d.ts`

```typescript
root
 --src
  --floder
   --moduleA.ts

// A.ts
import { b } from "./moduleB"
```

相对导入:文件路径

非相对路径:逐层遍历到根目录

##### Node

查找

1.  `moduleName.ts`
2. `moduleName.tsx`
3. `moduleName.d.ts` 
4. `moduleName/package.json=>types`
5. `moduleName/index.ts`
6. `moduleName/index.tsx`
7. `module/index.d.ts`

相对导入:文件路径

非相对路径:逐层遍历node_modules到根目录

##### 附加的模块解析标记

指导模块导入的额外信息

> tsconfig.json中配置

###### *Base URL*

所有非相对模块导入都会被当做相对于 `baseUrl`

###### *路径映射*

```json
{
  "compilerOptions": {
    "baseUrl": ".", // This must be specified if "paths" is.
    "paths": {
      "jquery": ["node_modules/jquery/dist/jquery"] // 此处映射是相对于"baseUrl"
    }
  }
}
```

注意`"paths"`是相对于`"baseUrl"`进行解析

```js
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "*": [ // 所有匹配"*"（所有的值）
        "*", // <moduleName> => <baseUrl>/<moduleName> // 
        "generated/*" // <moduleName> => <baseUrl>/generated/<moduleName>
      ]
    }
  }
}
```

###### *利用`rootDirs`指定虚拟目录*

列表里的内容会在运行时被合并

```js
{
  "compilerOptions": {
    "rootDirs": [
      "src/views",
      "generated/templates/views"
    ]
  }
}
```

##### 跟踪模块解析

`--traceResolution`

##### 使用`--noResolve`

告诉编译器不要添加任何不是在命令行上传入的文件到编译列表

##### exclude

如果你想利用 `“exclude”`排除某些文件，甚至你想指定所有要编译的文件列表，请使用`“files”`



---



## 17. 声明合并

声明合并”是指编译器将针对同一个名字的两个独立声明合并为单一声明。 合并后的声明同时拥有原先两个声明的特性。 任何数量的声明都可被合并；不局限于两个声明。

> 这不是一种良好的模式

### 基础概念

实体：

- namespace
- type
  - class
  - enum
  - interface
  - type alias
- value
  - namespace
  - class
  - Enum
  - function
  - Variable

### 合并接口

把双方的成员放到一个同名的接口里

- 非函数的成员应该是唯一
  - 如果不是唯一的，那么它们必须是相同的类型
- 每个同名函数声明都会被当成这个函数的一个重载
  - 后面的接口具有更高的优先级
  - 若参数的类型是 *单一*的字符串字面量，则提升到重载列表的最顶端

```typescript
interface Document {
    createElement(tagName: any): Element;
}
interface Document {
    createElement(tagName: "div"): HTMLDivElement;
    createElement(tagName: "span"): HTMLSpanElement;
}
interface Document {
    createElement(tagName: string): HTMLElement;
    createElement(tagName: "canvas"): HTMLCanvasElement;
}
// canvas div span 提升
interface Document {
    createElement(tagName: "canvas"): HTMLCanvasElement;
    createElement(tagName: "div"): HTMLDivElement;
    createElement(tagName: "span"): HTMLSpanElement;
    createElement(tagName: string): HTMLElement;
    createElement(tagName: any): Element;
}
```



### 合并命名空间

- 模块导出的同名接口进行合并
- 非导出成员仅在其原有的（合并前的）命名空间内可见



### 命名空间与类和函数和枚举类型合并

TypeScript使用这个功能去实现一些JavaScript里的设计模式

#### 合并命名空间和类

##### 内部类

```typescript
class Album {
    label: Album.AlbumLabel;
}
namespace Album {
    export class AlbumLabel { }
}
```

##### 扩展函数

```typescript
function buildLabel(name: string): string {
    return buildLabel.prefix + name + buildLabel.suffix;
}

namespace buildLabel {
    export let suffix = "";
    export let prefix = "Hello, ";
}

console.log(buildLabel("Sam Smith"))
```

##### 扩展枚举

```typescript
enum Color {
    red = 1,
    green = 2,
    blue = 4
}

namespace Color {
    export function mixColor(colorName: string) {
        if (colorName == "yellow") {
            return Color.red + Color.green;
        }
        else if (colorName == "white") {
            return Color.red + Color.green + Color.blue;
        }
        else if (colorName == "magenta") {
            return Color.red + Color.blue;
        }
        else if (colorName == "cyan") {
            return Color.green + Color.blue;
        }
    }
}
```



### 非法的合并

类不能与其它类或变量合并



### 模块扩展

使用declare声明模块扩展

使用interface扩展类

```typescript
// user.ts
export class User {
  login () {}
}
// index.ts
import { User } from '@/user'
declare module '@/user' {
  interface User {
    logout: () => void
  }
}
User.prototype.logout = function () {}
```

模块名的解析和用 `import`/ `export`解析模块标识符的方式是一致的

当这些声明在扩展中合并时，就好像在原始位置被声明了一样。

你不能在扩展中声明新的顶级声明－仅可以扩展模块中已经存在的声明。

#### 全局扩展

```typescript
declare global {
    interface Array<T> {
        toObservable(): Observable<T>;
    }
}
```



---



## 18.JSX

### 基本用法

1. 给文件一个`.tsx`扩展名
2. 启用`jsx`选项

| 模式           | 输入      | 输出                         | 输出文件扩展名 |
| :------------- | :-------- | :--------------------------- | :------------- |
| `preserve`     | `<div />` | `<div />`                    | `.jsx`         |
| `react`        | `<div />` | `React.createElement("div")` | `.js`          |
| `react-native` | `<div />` | `<div />`                    | `.js`          |

>*React标识符是写死的硬编码，所以你必须保证React（大写的R）是可用的*

### `as`操作符

TypeScript在`.tsx`文件里禁用了使用尖括号的类型断言

#### 类型检查

##### 固有元素

```typescript
declare namespace JSX {
    interface IntrinsicElements {
        div: any
    }
}
declare namespace JSX {
    interface IntrinsicElements {
        [elemName: string]: any;
    }
}  
```

##### 基于值的元素

作用域里按标识符查找

```typescript
import MyComponent from "./myComponent";
                                
<MyComponent />; // 正确
<SomeOtherComponent />; // 错误
```

1. 无状态函数组件 (SFC)
2. 类组件

*类组件*

扩展用来限制JSX的类型以符合相应的接口

```typescript
declare namespace JSX {
    interface ElementClass {
    	render: any;
    }
}
```

##### 属性类型检查

*固有元素*

```typescript
declare namespace JSX {
    interface IntrinsicElements {
    	foo: { bar?: boolean }
    }
}
<foo bar />;  
```

*基于值的元素*

1. JSX.ElementAttributesProperty指定
2. 类元素构造函数或SFC调用的第一个参数的类型

```typescript
declare namespace JSX {
    interface ElementAttributesProperty {
    	props; // 指定用来使用的属性名
    }
}
class MyComponent {
    // 在元素实例类型上指定属性
    props: {
    	foo?: string;
    }
}
// `MyComponent`的元素属性类型为`{foo?: string}`
<MyComponent foo="bar" />
```

- 如果一个属性名不是个合法的JS标识符（像`data-*`属性），并且它没出现在元素属性类型里时不会当做一个错误。
- 使用`JSX.IntrinsicAttributes`接口来指定额外的属性，这些额外的属性通常不会被组件的props或arguments使用 - 比如React里的`key`
  - 支持`JSX.IntrinsicClassAttributes<T>`
    - `Ref<T>`

##### 子孙类型检查

使用JSX.ElementAttributesProperty声明props

SFC使用`JSX.ElementChildrenAttribute`声明 children

```typescript
declare namespace JSX {
    interface ElementChildrenAttribute {
    	children: {};  // specify children name to use
    }
}
const CustomComp = (props) => <div>{props.children}</div>


```

类组件使用`PropsType`声明children

```typescript
interface PropsType {
    children: JSX.Element
    name: string
}

class Component extends React.Component<PropsType, {}> {
    render() {
        return (
            <h2>
            {this.props.children}
            </h2>
        )
    }
}
```



### JSX结果类型

默认地JSX表达式结果的类型为`any`

你可以自定义这个类型，通过指定`JSX.Element`接口。 然而，不能够从接口里检索元素，属性或JSX的子元素的类型信息。 它是一个黑盒。



### React整合

```typescript
/// <reference path="react.d.ts" />

interface Props {
    foo: string;
}

class MyComponent extends React.Component<Props, {}> {
    render() {
    	return <span>{this.props.foo}</span>
    }
}
```



### 工厂函数

`jsx: react`编译选项使用的工厂函数是可以配置的。可以使用`jsxFactory`命令行选项，或内联的`@jsx`注释指令在每个文件上设置

```typescript
import preact = require("preact");
/* @jsx preact.h */
const x = <div />;
```

工厂函数的选择同样会影响`JSX`命名空间的查找（类型检查）

> 本章需要实际应用一下



## 19. 装饰器

能够被附加到[类声明](https://www.tslang.cn/docs/handbook/decorators.html#class-decorators)，[方法](https://www.tslang.cn/docs/handbook/decorators.html#method-decorators)， [访问符](https://www.tslang.cn/docs/handbook/decorators.html#accessor-decorators)，[属性](https://www.tslang.cn/docs/handbook/decorators.html#property-decorators)或[参数](https://www.tslang.cn/docs/handbook/decorators.html#parameter-decorators)上

在运行时被调用

#### 装饰器组合

多个装饰器可以同时应用到一个声明上

```typescript
@f
@g
x
```

(*f* ∘ *g*)(*x*)等同于*f*(*g*(*x*))

1. 由上至下依次对装饰器表达式求值。
2. 求值的结果会被当作函数，由下至上依次调用。

*执行顺序*

```typescript
function f() {
    console.log("f(): evaluated");
    return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("f(): called");
    }
}

function g() {
    console.log("g(): evaluated");
    return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("g(): called");
    }
}

class C {
    @f()
    @g()
    method() {}
}

// f(): evaluated
// g(): evaluated
// g(): called
// f(): called
```

#### 装饰器求值

1. *参数装饰器*，然后依次是*方法装饰器*，*访问符装饰器*，或*属性装饰器*应用到每个实例成员。
2. *参数装饰器*，然后依次是*方法装饰器*，*访问符装饰器*，或*属性装饰器*应用到每个静态成员。
3. *参数装饰器*应用到构造函数。
4. *类装饰器*应用到类。

#### 类装饰器

类装饰器不能用在声明文件中( `.d.ts`)，也不能用在任何外部上下文中（比如`declare`的类）

类的构造函数作为其唯一的参数

```typescript
function classDecorator(constructor){
  constructor
  constructor.prototype
  return class 
}
```

> 修改构造器和原型
>
> 使用返回直接修改类重载构造函数

#### 方法装饰器

监视，修改或者替换方法定义

方法装饰器表达式会在运行时当作函数被调用，传入下列3个参数：

1. 对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。
   - 因为静态成员挂载在构造函数，而共享的实例方法挂载在原型对象上
2. 成员的名字。
3. 成员的*属性描述符*。

如果方法装饰器返回一个值，它会被用作方法的*属性描述符*

```typescript
function methodDecorator(constructor,propertyKey:string,descriptor:PropertyDescriptor){
  
}
```

#### 访问器装饰器

1. 对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。
2. 成员的名字。
3. 成员的*属性描述符*。

不允许同时装饰一个成员的`get`和`set`访问器

> 也没必要

#### 属性装饰器

*属性装饰器*声明在一个属性声明之前（紧靠着属性声明）。 属性装饰器不能用在声明文件中（.d.ts），或者任何外部上下文（比如 `declare`的类）里。

1. 对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。
2. 成员的名字。

*属性描述符*不会做为参数传入属性装饰器

> 需要调用构造器才能生成实例属性
>
> 可以针对属性名存储一些信息，在运行时再通过属性名获取

#### 参数装饰器

*参数装饰器*声明在一个参数声明之前（紧靠着参数声明）。 参数装饰器应用于类构造函数或方法声明。 参数装饰器不能用在声明文件（.d.ts），重载或其它外部上下文（比如 `declare`的类）里。

参数：

1. 对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。
2. 成员的名字。
3. 参数在函数参数列表中的索引。

> 参数修饰器只能监视参数是否在方法中被声明，且返回值会被忽略
>
> 可以针对参数位置存储一些信息， 在运行时通过参数index获取相关的信息
>
> 配合方法装饰器使用

#### 元数据

设计阶段添加的类型信息可以在运行时使用

```typescript
let type = Reflect.getMetadata("design:type", target, propertyKey)
```

```typescript
class Line {
    private _p0: Point;
    private _p1: Point;

    @validate
    @Reflect.metadata("design:type", Point)
    set p0(value: Point) { this._p0 = value; }
    get p0() { return this._p0; }

    @validate
    @Reflect.metadata("design:type", Point)
    set p1(value: Point) { this._p1 = value; }
    get p1() { return this._p1; }
}

```

`emitDecoratorMetadata`开启自动注入修饰器

> 运行时属性校验和依赖注入皆依赖于此特性



---



## 20. Mixin

1. implements需要继承的类
2. 把父类当成接口，声明其方法和属性（只需最简单的实现，占位用即可）
3. 混入原型

```typescript
class SmartObject implements Disposable, Activatable {
    constructor() {
        setInterval(() => console.log(this.isActive + " : " + this.isDisposed), 500);
    }

    interact() {
        this.activate();
    }

    // Disposable
    isDisposed: boolean = false;
    dispose: () => void;
    // Activatable
    isActive: boolean = false;
    activate: () => void;
    deactivate: () => void;
}
applyMixins(SmartObject, [Disposable, Activatable]);

let smartObj = new SmartObject();
setTimeout(() => smartObj.interact(), 1000);

////////////////////////////////////////
// In your runtime library somewhere
////////////////////////////////////////

function applyMixins(derivedCtor: any, baseCtors: any[]) {
    baseCtors.forEach(baseCtor => {
        Object.getOwnPropertyNames(baseCtor.prototype).forEach(name => {
            derivedCtor.prototype[name] = baseCtor.prototype[name];
        });
    });
}
```

> implements只负责声明，不包括具体的mixin实现



---



## 21. 三斜线指令

- 包含单个XML标签的单行注释
- *仅*可放在包含它的文件的最顶端

### path

`/// <reference path="..." />`

用于声明文件间的依赖

##### *预处理输入文件*

引用路径是相对于包含它的文件的，如果不是根文件

##### *错误*

引用不存在的文件会报错。 一个文件用三斜线指令引用自己会报错。

##### *`--noResolve`*

三斜线引用会被忽略



### types

声明了对某个包的依赖

对这些包的名字的解析与在 `import`语句里对模块名的解析类似。 

可以简单地把三斜线类型引用指令当做 `import`声明的包。

```typescript
/// <reference types="node" />
// 表明这个文件使用了 @types/node/index.d.ts里面声明的名字
// 并且，这个包需要在编译阶段与声明文件一起被包含进来
```

仅当在你需要写一个`d.ts`文件时才使用这个指令

对于那些在编译阶段生成的声明文件，编译器会自动地添加`/// <reference types="..." />`； *当且仅当*结果文件中使用了引用的包里的声明时才会在生成的声明文件里添加`/// <reference types="..." />`语句。

若要在`.ts`文件里声明一个对`@types`包的依赖，使用`--types`命令行选项或在`tsconfig.json`里指定。 查看 [在`tsconfig.json`里使用`@types`，`typeRoots`和`types`](https://www.tslang.cn/docs/handbook/tsconfig-json.html#types-typeroots-and-types)了解详情。

> 对外发布包需要用到



### lib

指定内置lib文件

和*tsconfig.json*中的lib选项功能一致



### no-default-lib

把一个文件标记成*默认库*

##### *--noLib*

告诉编译器在编译过程中*不要*包含这个默认库（比如，`lib.d.ts`）

##### *--skipDefaultLibCheck*

忽略检查带有`/// <reference no-default-lib="true"/>`的文件



### amd-module

```typescript
///<amd-module name='NamedModule'/>
```

给编译器传入一个可选的模块名, 当成amd define的模块名



---



## 22. JavaScript文件类型检查

> 不如直接ts吧

`--checkJs`对`.js`文件进行类型检查和错误提示

##### 注释

```javascript
// @ts-nocheck 忽略类型检查

// @ts-check 去掉--checkJs设置并添加注释来选则检查某些.js文件

// @ts-ignore 忽略本行的错误
```

严格检查标记，如`noImplicitAny`，`strictNullChecks`表现可能会相对宽松

### 差异

##### 1. 用JSDoc类型表示类型信息

```javascript
/** @type {number} */
var x;
```

##### 2. 属性的推断来自于类内的赋值语句

对构造器属性使用注释声明或者类型断言，其他赋值则使用联合类型兼容

##### 3. 构造函数等同于类

es5

##### 4. 支持CommonJS模块

##### 5. 类，函数和对象字面量是命名空间

直接给函数或者类添加属性作为静态属性或者命名空间

##### 6. 对象字面量是开放的

对象字面量的表现就好比具有一个默认的索引签名`[x:string]: any`

##### 7. null，undefined初始化的类型是any,空数组的类型为any[]

##### 8. 函数参数是默认可选的

使用JSDoc注解的函数会被从这条规则里移除。 使用JSDoc可选参数语法来表示可选性

```javascript
/**
 * @param {string} [somebody] - Somebody's name.
 */
function sayHello(somebody) {}
```

##### 9. 由`arguments`推断出的var-args参数声明

```javascript
/** @param {...number} args */
function sum(/* numbers */) {}
```

##### 10. 未指定的类型参数默认为`any`

*在extends语句中：*

```typescript
// @augments来明确地指定类型
/**
 * @augments {Component<{a: number}, State>}
 */
```

*在JSDoc引用中：*

```javascript
// 类型数组
/** @type{Array.<number>} */
var y = [];
```

*在函数调用中*

```javascript
var p = new Promise((resolve, reject) => { reject() });
p; // Promise<any>;
```



### 支持的JSDoc

- `@type`
- `@param` (or `@arg` or `@argument`)
- `@returns` (or `@return`)
- `@typedef`
- `@callback`
- `@template`
- `@class` (or `@constructor`)
- `@this`
- `@extends` (or `@augments`)
- `@enum`

#### `@type`

*转换*

```javascript
/**
 * @type {number | string}
 */
var numberOrString = Math.random() < 0.5 ? "hello" : 100;
var typeAssertedNumber = /** @type {number} */ (numberOrString)
```

*导入类型*

```javascript
/**
 * @param p { import("./a").Pet }
 */
function walk(p) {
    console.log(`Walking ${p.name}...`);
}
// 值类型提取
/**
 * @type {typeof import("./a").x }
 */
var x = require("./a").x;
```

#### `@param`和`@returns`

*可选参数*

```javascript
// Parameters may be declared in a variety of syntactic forms
/**
 * @param {string}  p1 - A string param.
 * @param {string=} p2 - An optional param (Closure syntax)
 * @param {string} [p3] - Another optional param (JSDoc syntax).
 * @param {string} [p4="test"] - An optional param with a default value // 推荐使用
 * @return {string} This is the result
 */
function stringsStringStrings(p1, p2, p3, p4){
  // TODO
}
```

#### `@typedef`, `@callback`, 和 `@param`

`@typedef`可以用来声明复杂类型

```javascript
/**
 * @typedef {Object} SpecialType - creates a new type named 'SpecialType'
 * @property {string} prop1 - a string property of SpecialType
 * @property {number} prop2 - a number property of SpecialType
 * @property {number=} prop3 - an optional number property of SpecialType
 * @prop {number} [prop4] - an optional number property of SpecialType
 * @prop {number} [prop5=42] - an optional number property of SpecialType with default
 */
/** @type {SpecialType} */
var specialTypeObject;
```

> 远离这玩意儿，ts一行代码搞定

#### `@template`

使用`@template`声明泛型：

> 魔鬼

支持类型约束

#### `@constructor`

构造函数调用检查

#### `@this`

明确指定this

#### `@extends`

```javascript
/**
 * @template T
 * @extends {Set<T>}
 */
class SortableSet extends Set {
  // ...
}
// 注意@extends只作用于类。当前，无法实现构造函数继承类的情况。
```

#### `@enum`

### 不支持的模式

- 值空间中将对象视为类型
- Nullable(只在启用了`strictNullChecks`检查时才启作用)
- Non-nullable(同上)

> 不推荐使用



---



## 23. 通用类型

#### `Partial<T>`

将T的所有属性设为可选的

#### `Readonly<T>`

将T的所有属性设为只读的

#### `Record<K,T>`

Map K（通常是联合类型）的属性（实用in操作符），属性类型为T

#### `Pick<T,K>`

提取T中的K（通常是联合类型）的属性类型

#### `Omit<T,K>`

与pick相反

#### `Exclude<T,U>`

排除U类型

T,U通常都是联合类型

#### `Extract<T,U>`

U为指定类型 提取T中符合U类型的属性类型

T通常是联合类型

#### `NonNullable<T>`

排除nul l和undefiend类型

#### `ReturnType<T>`

T为函数类型

#### `InstanceType<T>`

T为类类型

#### `Required<T>` 

将T的所有属性设为必须的

#### `ThisType<T>`

不返回类型，用于指定上下文this类型