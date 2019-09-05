# typescript

## 基础类型

### 元组

- 混合数组

### 枚举

### void

### Null ,Undefined

### Any

### Never

### 类型断言

- 范型

	- 尖括号

- as操作符

## 变量声明

### 函数声明

- 解构

	- 注意默认值设置

### type

- union和元组常用
- 不能扩展

## 接口

### 鸭式辨型法

### 可选属性

### 只读属性

- ReadonlyArray<T>

	- 赋值给Array需要类型断言

### 额外的属性检查

- 索引签名

	- [propName: string]: any;

### 函数类型

- (source: string, subString: string): boolean;

### 可索引的类型

- 使用 number来索引时，JavaScript会将它转换成string然后再去索引对象

	- TypeScript支持两种索引签名：字符串和数字。 可以同时使用两种类型的索引，但是数字索引的返回值必须是字符串索引返回值类型的子类型

- dictionary模式

	- 字符串索引签名

- 索引签名可设置为只读

### 类类型

- 实现接口
- 类静态部分与实例部分的区别

	- 当你用构造器签名去定义一个接口并试图定义一个类去实现这个接口时会得到一个错误

- 继承接口

	- 接口可以继承多个接口
- 混合类型

	- 一个对象可以同时做为函数和对象使用，并带有额外的属性
- 接口继承类

	- 接口会继承到类的private和protected成员。 这意味着当你创建了一个接口继承了一个拥有私有或受保护的成员的类时，这个接口类型只能被这个类或其子类所实现
	- 当你有一个庞大的继承结构时这很有用，但要指出的是你的代码只在子类拥有特定属性时起作用。 这个子类除了继承至基类外与基类没有任何关系

## 类

### 继承

- 在构造函数里访问 this的属性之前， 一定要调用 super()

### 公共，私有与受保护的修饰符

- 如果其中一个类型里包含一个 private成员，那么只有当另外一个类型中也存在这样一个 private成员， 并且它们都是来自同一处声明时，我们才认为这两个类型是兼容的。
- protected成员在派生类中仍然可以访问
- 构造函数也可以被标记成 protected。 这意味着这个类不能在包含它的类外被实例化，但是能被继承。

### readonly修饰符

-  只读属性必须在声明时或构造函数里被初始化

### 参数属性

- 参数属性可以方便地在一个地方定义并初始化一个成员

### 只带有 get不带有 set的存取器自动被推断为 readonly

### 抽象类

-  不同于接口，抽象类可以包含成员的实现细节
- 抽象方法必须包含 abstract关键字并且可以包含访问修饰符

### 高级技巧

- 构造函数

	- typeof可以获取构造函数的类型

- 把类当做接口使用

## 函数

### 书写完整函数类型

- 包含参数类型和返回值类型

	- 每个参数指定一个名字和类型。 这个名字只是为了增加可读性
	- 使用( =>)符号标识返回值类型

### 可选参数

- （?）标识

### this

- this参数

	- 提供一个显式的 this参数。 this参数是个假的参数，它出现在参数列表的最前面
	- this参数在回调函数里

		- 库函数的作者要指定 this的类型

### 重载

- 为同一个函数提供多个函数类型定义来进行函数重载
- 在定义重载的时候，一定要把最精确的定义放在最前面

## 范型

### 泛型变量

- Array<T>
- T[]

### 泛型类型

- 泛型函数的类型与非泛型函数的类型没什么不同，只是有一个类型参数在最前面

  function identity<T>(arg: T): T {
      return arg;
  }

- 泛型接口

	- 对象字面量形式

	  interface GenericIdentityFn {
	      <T>(arg: T): T;
	  }

	- 传入一个类型参数来指定泛型类型

	  interface GenericIdentityFn<T> {
	      (arg: T): T;
	  }

### 泛型类

class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

- 类的静态属性不能使用这个泛型类型

### 泛型约束

- extends实现约束

  interface Lengthwise {
      length: number;
  }
  
  function loggingIdentity<T extends Lengthwise>(arg: T): T {
      console.log(arg.length);
      return arg;
  }

- 在泛型约束中使用类型参数

	- keyof

	  function getProperty<T, K extends keyof T>(obj: T, key: K) {
	      return obj[key];
	  }
	  
	  let x = { a: 1, b: 2, c: 3, d: 4 };
	  
	  getProperty(x, "a"); // okay
	  getProperty(x, "m"); // error

	- 在泛型里使用类类型

	  function create<T>(c: {new(): T; }): T {
	      return new c();
	  }
	  function createInstance<A extends Animal>(c: new () => A): A {
	      return new c();
	  }

## 枚举

### 数字枚举自增

- 可指定初始值

### 通过枚举的属性来访问枚举成员，和枚举的名字来访问枚举类型

### 使用计算值时需要初始化

### 在一个字符串枚举里，每个成员都必须用字符串字面量，或另外一个字符串枚举成员进行初始化

### 联合枚举与枚举成员的类型

- 枚举成员可以成为，说明成员只能是枚举成员的值
- 联合枚举
- 运行时的枚举

	- 枚举是在运行时真正存在的对象

- 反向映射

	- 数字枚举成员具有反向映射，从枚举值到枚举名字

### const枚举

- 常量枚举只能使用常量枚举表达式，并且不同于常规的枚举，它们在编译阶段会被删除

### 外部枚举

- 外部枚举用来描述已经存在的枚举类型的形状

	- 非常数的外部枚举，没有初始化方法时被当做需要经过计算的

*XMind: ZEN - Trial Version*