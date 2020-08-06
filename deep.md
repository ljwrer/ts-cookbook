# 深入理解 TypeScript

# typescript学习相关文档

官方文档: <https://www.typescriptlang.org/docs/handbook/basic-types.html>  
中文版: <https://www.tslang.cn/docs/handbook/basic-types.html>  

《深入理解 TypeScript》  
原版: <https://basarat.gitbook.io/typescript/>  
中文版: <https://jkchao.github.io/typescript-book-chinese/>

## 关键字

- declare： 环境声明
- is： 自定义类型保护
- 范型
  - extends
    - `<T extends string>`
  - in
    - `{ [K in T]: K }`

# **TypeScript 项目**

# 1. 模块

## 全局模块

没有import export关键字



## 文件模块

含有 `import` 或者 `export`

本地作用域



## 文件模块详情

> 使用 module: commonjs

### ES 模块语法

#### 默认导入／导出

> 不要使用默认导出



# 2. 命名空间

对大多数项目来说，我们推荐使用一个使用 `namespace` 的外部的模块，用来快速的演示和移植旧的 JavaScript 代码。



# **TypeScript 类型系统**

# 1. 概览

##  特殊类型

### void

使用 `void` 来表示一个函数没有一个返回值

## 类型别名

>TIP
>
>- 如果你需要使用类型注解的层次结构，请使用接口。它能使用 `implements` 和 `extends`
>- 为一个简单的对象类型使用类型别名，仅仅有一个语义化的作用。与此相似，当你想给一个联合类型和交叉类型使用一个语意化的名称时，一个类型别名将会是一个好的选择。



# 2. 从 JavaScript 迁移

- 添加一个 `tsconfig.json` 文件；
- 把文件扩展名从 `.js` 改成 `.ts`，开始使用 `any` 来减少错误；
- 开始在 TypeScript 中写代码，尽可能的减少 `any` 的使用；
- 回到旧代码，开始添加类型注解，并修复已识别的错误；
- 为你的第三方 JavaScript 代码定义环境声明。

## 额外的非 JavaScript 资源

你只要添加如下代码（放在 `globals.d.ts`）

```typescript
declare module '*.css';
```



# 3. @types

## 控制全局

通过配置 `compilerOptions.types: [ "jquery" ]` 后，只允许使用 `jquery` 的 `@types` 包，即使这个人安装了另一个声明文件，比如 `npm install @types/node`，它的全局变量（例如 `process`）也不会泄漏到你的代码中，直到你将它们添加到 tsconfig.json 类型选项



# 4. 环境声明

## 声明文件

你可以通过 `declare` 关键字，来告诉 TypeScript，你正在试图表述一个其他地方已经存在的代码

你可以选择把这些声明放入 `.ts` 或者 `.d.ts` 里。在你实际的项目里，我们强烈建议你应该把声明放入 `.d.ts` 里（可以从一个命名为 `globals.d.ts` 或者 `vendor.d.ts` 文件开始）。

如果一个文件有扩展名 `.d.ts`，这意味着每个顶级的声明都必须以 `declare` 关键字作为前缀。这有利于向作者说明，在这里 TypeScript 将不会把它编译成任何代码，同时他需要确保这些在编译时存在。

>TIP
>
>- 环境声明就好像你与编译器之间的一个约定，如果这些没有在编译时存在，但是你却使用了它们，则事情将会在没有警告的情况下中断。
>- 环境声明就好像是一个文档。如果源文件更新了，你应该同步更进。所以，当你使用源文件在运行时的新行为时，如果没有人更新环境声明，编译器将会报错。

## 变量

推荐尽可能的使用接口(便于扩展，接口自动合并)

```ts
interface Process {
  exit(code?: number): void;
}

declare let process: Process;
```

#  5. 接口

接口运行时的影响为 0

因为 TypeScript 接口是开放式的，这是 TypeScript 的一个重要原则，它允许你使用接口模仿 JavaScript 的可扩展性。

类是具有两个类型的：静态部分的类型和实例的类型

# 6. 枚举

##  改变与数字枚举关联的数字

>TIP
>
>我通常用 `= 1` 初始化，因为在枚举类型值里，它能让你做一个安全可靠的检查。

## 使用数字类型作为标志

> 除非你真的理解了，谨慎使用位运算

- 我们使用 `|=` 来添加一个标志；
- 组合使用 `&=` 和 `~` 来清理一个标志；
- `|` 来合并标志

# 7. `lib.d.ts`

- 它自动包含在 TypeScript 项目的编译上下文中；
- 它能让你快速开始书写经过类型检查的 JavaScript 代码。

## 修改原始类型

```typescript
// 确保是模块
export {};
// 声明全局扩展
declare global {
  interface String {
    endsWith(suffix: string): boolean;
  }
}
// 实现声明
String.prototype.endsWith = function(suffix: string): boolean {
  const str: string = this;
  return str && str.indexOf(suffix, str.length - suffix.length) !== -1;
};
```

## 编译目标对 `lib.d.ts` 的影响

设置编译目标为 `es6` 时，能导致 `lib.d.ts` 包含更多的像 Promise 的现代（es6）内容的环境声明。编译器目标的这种神奇作用，改变了代码的环境

## `--lib` 选项

> 使用 `--lib` 选项可以将任何 `lib` 与 `--target` 解偶。

>`--lib` 选项提供非常精细的控制，因此你最有可能从运行环境与 JavaScript 功能类别中分别选择一项，如果你没有指定 `--lib`，则会导入默认库：
>
>- `--target` 选项为 es5 时，会导入 es5, dom, scripthost。
>- `--target` 选项为 es6 时，会导入 es6, dom, dom.iterable, scripthost。

推荐：

```json
"compilerOptions": {
  "target": "es5",
  "lib": ["es6", "dom"]
}
```

包括使用 Symbol 的 ES5 使用例子：

```json
"compilerOptions": {
  "target": "es5",
  "lib": ["es5", "dom", "scripthost", "es2015.symbol"]
}
```

## 在旧的 JavaScript 引擎时使用 Polyfill

```ts
import 'core-js';
```



# 8. 函数

### 函数声明

```typescript
type LongHand = {
  (a: number): number;
};

type ShortHand = (a: number) => number;
// 当你想使用函数重载时，只能用第一种方式
type LongHandAllowsOverloadDeclarations = {
  (a: number): number;
  (a: string): string;
};
```



# 9. 可调用的

括号表示可调用的

```typescript
interface Overloaded {
  (foo: string): string;
  (foo: number): number;
}

type Overloaded {
  (foo: string): string;
  (foo: number): number;
}

const simple: (foo: number) => string = foo => foo.toString();

let overloaded: {
  (foo: string): string;
  (foo: number): number;
};
```

## 可实例化

new ()表示可实例化

```typescript
interface CallMeWithNewToGetString {
  new (): string;
}
```



# 10. 类型断言

为避免与 JSX 的语法存在歧义，建议使用 `as TypeAnnotation` 的语法来为类型断言

## 类型断言与类型转换

它之所以不被称为「类型转换」，是因为转换通常意味着某种运行时的支持。但是，类型断言纯粹是一个编译时语法，同时，它也是一种为编译器提供关于如何分析代码的方法。

## 类型断言被认为是有害的

如果你没有按约定添加属性，TypeScript 编译器并不会对此发出错误警告

如果你忘记了某个属性，编译器同样也不会发出错误警告

```ts
interface Foo {
  bar: number;
  bas: string;
}

const foo = {} as Foo; // bad
const foo: Foo = {
  // 编译器将会提供 Foo 属性的代码提示
}; // good
```

## 双重断言

```typescript
function handler(event: Event) {
  const element = (event as any) as HTMLElement; // ok
}
```

> 3.x 版本以后使用unknown替代any

### TypeScript 是怎么确定单个断言是否足够

```
if(s<<T || T << S) then S as T
// << 表示类型收敛
```





#  11. Freshness

更严格的对象字面量检查

```typescript
function logName(something: { name: string }) {
  console.log(something.name);
}

logName({ name: 'matt' }); // ok
logName({ name: 'matt', job: 'being awesome' }); // Error: 对象字面量只能指定已知属性，`job` 属性在这里并不存在
let info = { name: 'matt', job: 'being awesome' } 
logName(info) // ok
```

之所以只对对象字面量进行类型检查，因为在这种情况下，那些实际上并没有被使用到的属性有可能会拼写错误或者会被误用。

## 允许额外的属性

使用索引签名

### 缺省属性

使用可选属性



# 12. 类型保护

## 使用定义的类型保护

```ts
function isFoo(arg: Foo | Bar): arg is Foo {
  return (arg as Foo).foo !== undefined;
}
```



# 13. 字面量类型

## 字符串字面量

在一个联合类型中组合创建一个强大的（实用的）抽象

```typescript
type CardinalDirection = 'North' | 'East' | 'South' | 'West';
```

## 其他字面量类型

```typescript
type OneToFive = 1 | 2 | 3 | 4 | 5;
type Bools = true | false;
```

## 推断

使用类型断言或类型注解



# 14. readonly

函数参数，interface， type，class

### 映射类型

```ts
type Foo = {
  bar: number;
  bas: number;
};

type FooReadonly = Readonly<Foo>;
```

### 绝对的不可变

把索引签名标记为只读

```typescript
interface Foo {
  readonly [x: number]: number;
}
```

不可变数组

```typescript
let foo: ReadonlyArray<number> = [1, 2, 3];
```

## 与 `const` 的不同

```
const
```

- 用于变量；
- 变量不能重新赋值给其他任何事物。

```
readonly
```

- 用于属性；
- 用于别名，可以修改属性；

`readonly` 能确保“我”不能修改属性，但是当你把这个属性交给其他并没有这种保证的使用者（允许出于类型兼容性的原因），他们能改变它。

> 3.x以后可以使用as const

# 15. 泛型

>你可以随意调用泛型参数，当你使用简单的泛型时，泛型常用 `T`、`U`、`V` 表示。如果在你的参数里，不止拥有一个泛型，你应该使用一个更语义化名称，如 `TKey` 和 `TValue` （通常情况下，以 `T` 作为泛型的前缀，在其他语言如 C++ 里，也被称为模板）

##  设计模式：方便通用

仅使用一次的泛型并不比一个类型断言来的安全

### 配合 axios 使用

通常情况下，我们会把后端返回数据格式单独放入一个 interface 里



# 16. 类型推断

## 警告

### 小心使用返回值

尽管 TypeScript 一般情况下能推断函数的返回值，但是它可能并不是你想要的

> 我发现最简单的方式是明确的写上函数返回值，毕竟这些注解是一个定理，而函数是注解的一个证据。
>
> 这里值得商榷，当类型比较复杂时使用类型推断能极大提升开发速度

### `noImplicitAny`

选项 `noImplicitAny` 用来告诉编译器，当无法推断一个变量时发出一个错误（或者只能推断为一个隐式的 `any` 类型），你可以：

- 通过显式添加 `any` 的类型注解，来让它成为一个 `any` 类型；
- 通过一些更正确的类型注解来帮助 TypeScript 推断类型。



#  17. 类型兼容性

## 变体

- 协变（Covariant）：只在同一个方向；
- 逆变（Contravariant）：只在相反的方向；
- 双向协变（Bivariant）：包括同一个方向和不同方向；
- 不变（Invariant）：如果类型不完全相同，则它们是不兼容的。

> 此处请参考下文中的协变与逆变

## 函数

###  返回类型

协变（Covariant）：返回类型必须包含足够的数据。

### 参数数量

更少的参数数量是好的

### 可选的和 rest 参数

可选的（预先确定的）和 Rest 参数（任何数量的参数）都是兼容的：

> 可选的（上例子中的 `bar`）与不可选的（上例子中的 `foo`）仅在选项为 `strictNullChecks` 为 `false` 时兼容。

### 函数参数类型

双向协变（Bivariant）：旨在支持常见的事件处理方案。

> 需要类型断言

## 枚举

- 枚举与数字类型相互兼容
- 来自于不同枚举的枚举变量，被认为是不兼容的

## 类

- 仅仅只有实例成员和方法会相比较，构造函数和静态成员不会被检查。
- 私有的和受保护的成员必须来自于相同的类。

## 泛型

TypeScript 类型系统基于变量的结构，仅当类型参数在被一个成员使用时，才会影响兼容性。

如果尚未实例化泛型参数，则在检查兼容性之前将其替换为 `any`：

## 不变性

类型的协变和逆变容易导致未知的错误，保持不变最佳

```ts
let animalArr: Animal[] = [animal];
let catArr: Cat[] = [cat];
animalArr = catArr // 协变，ok但危险
animalArr.push(new Animal())
catArr.forEach(cat => cat.meow())
```



# 18. never

`never` 类型是 TypeScript 中的底层类型

`any` 类型是 TypeScript 中的底层类型和顶层类型

`never` 类型仅能被赋值给另外一个 `never`

## 详细的检查

never类型用来兜底

## void

`void` 表示没有任何类型，`never` 表示永远不存在的值的类型



# 19. 辨析联合类型

```ts
function area(s: Shape) {
  switch (s.kind) {
    case 'square':
      return s.size * s.size;
    case 'rectangle':
      return s.width * s.height;
    case 'circle':
      return Math.PI * s.radius ** 2;
    default:
      const _exhaustiveCheck: never = s;
      return _exhaustiveCheck;
  }
}
```

## Redux

```ts
type Action =
  | {
      type: 'INCREMENT';
    }
  | {
      type: 'DECREMENT';
    };

function counter(state = 0, action: Action) {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1;
    case 'DECREMENT':
      return state - 1;
    default:
      return state;
  }
}
```



#  20. 索引签名

## TypeScript 索引签名

强制用户必须明确的写出 `toString()` 的原因是：在对象上隐式执行的 `toString` 方法是有害的

TypeScript 的索引签名必须是 `string` 或者 `number`。

`symbols` 也是有效的，TypeScript 支持它。



## 所有成员都必须符合字符串的索引签名

## 使用一组有限的字符串字面量

```ts
type Index = 'a' | 'b' | 'c';
type FromIndex = { [k in Index]?: number };
```

通常与 `keyof/typeof` 一起使用，来获取变量的类型

```ts
type FromSomeIndex<K extends string> = { [key in K]: number };
```

## 同时拥有 `string` 和 `number` 类型的索引签名

`string` 类型的索引签名比 `number` 类型的索引签名更严格

```ts
interface ArrStr {
  [key: string]: string | number; // 必须包括所用成员类型
  [index: number]: string; // 字符串索引类型的子级

  // example
  length: number;
}
```

## 设计模式：索引签名的嵌套

尽量不要使用这种把字符串索引签名与有效变量混合使用

取而代之，我们把索引签名分离到自己的属性里，如命名为 `nest`（或者 `children`、`subnodes` 等）：

```ts
interface NestedCSS {
  color?: string;
  nest?: {
    [selector: string]: NestedCSS;
  };
}
```

## 索引签名中排除某些属性

交叉类型可以解决属性合并至索引签名的问题

```ts
type FormState = { isValid: boolean } & { [fieldName: string]: FieldState };
```

但不能显示创建该类型的对象



# 21. 流动的类型

> 此节是灵活使用typescript的关键！

## 复制类型和值

namespace + import

```ts
namespace scope{
  export class Foo{}
}
import Bar = scope.Foo
```

## 捕获变量的类型

typeof

## 捕获类成员的类型及方法返回值类型

declare+typeof

```ts
class Foo{
  bar: Bar
  run(){}
}
declare let _foo: Foo
const baz: typeof _foo.bar
// 捕获方法返回值
type RetOfRun = ReturnType<Foo['run']>
```

## 捕获键的名称

keyof typeof

```ts
const colors = {
  red: 0,
  blue: 1,
  green: 3
}
const colorName: keyof typeof colors = 'red'
```



# 22. 异常处理

## 使用 `Error`

不要抛出原始字符串

使用 `Error` 对象的基本好处是，它能自动跟踪堆栈的属性构建以及生成位置

## 你并不需要 `throw` 抛出一个错误

callback风格的错误处理不需要throw

## 优秀的用例

「Exceptions should be exceptional」

通过为每个可能抛出错误的代码显式捕获，来使其优雅

> 除非你想用以非常通用（try/catch）的方式处理错误，否则不要抛出错误
>
> 推荐使用try catch



# 23. 混合

> react的反向继承也是类似的思路

*定义构造器类型*

```ts
type Constructor<T = {}> = new (...args: any[]) => T;
```

```ts
// TBase必须是一个构造器
function TimesTamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    timestamp = Date.now();
  };
}
```



# 24. ThisType

```ts
type ObjectDescriptor<D, M> = {
  data?: D;
  methods?: M & ThisType<D & M>; // Type of 'this' in methods is D & M
};
```

此处ThisType指定了method的this类型为 D和M的交叉类型

```ts
function makeObject<D, M>(desc: ObjectDescriptor<D, M>): D & M {
  let data: object = desc.data || {};
  let methods: object = desc.methods || {};
  return { ...data, ...methods } as D & M;
}
```

`ThisType` 的接口，在 `lib.d.ts` 只是被声明为空的接口，除了可以在对象字面量上下文中可以被识别以外，该接口的作用等同于任意空接口。

> 推荐只在对象字面了使用
>
> 推荐只在class内使用this



# JSX

使用动机

- 同时检查js和html
- 让视图层了解运行时上下文（加强控制器和视图连接）
- 复用 JavaScript 设计模式维护 HTML 

# React JSX

> 优先使用类型推断

## 类型检查

###  HTML 标签

```ts
declare namespace JSX {
  interface IntrinsicElements {
    a: React.HTMLAttributes;
    abbr: React.HTMLAttributes;
    div: React.HTMLAttributes;
    span: React.HTMLAttributes;

    // 其他
  }
}
```

### 组件

```tsx
// 函数式组件
React.FunctionComponent<Props>
// 类组件
React.Component<Props,State> = 
React.ComponentClass<P> +
React.StatelessComponent<P>
// 组件的实例
React.ReactElement<T>
// render
React.ReactNode = JSX + string + null
```

### 泛型函数

```ts
const foo = <T extends {}>(x: T) => x;
```

### 强类型的 Refs

```ts
class Example extends React.Component {
  example() {
    // ... something
  }

  render() {
    return <div>Foo</div>;
  }
}
class Use {
  exampleRef: Example | null = null;

  render() {
    return <Example ref={exampleRef => (this.exampleRef = exampleRef)} />;
  }
}
```



# 解读 Errors

TypeScript 错误信息分为两类：简洁和详细



---



# tips

# 名义化类型

语义化名字的字符串

##  使用字面量类型

```ts
type Id<T extends string> = {
  type: T;
  value: string;
};
```

有额外的结构

## 使用枚举

```ts
enum FooIdBrand {
  _ = '' //推断为字符串枚举
}
type FooId = FooIdBrand & string;
```

## 使用接口

```ts
// FOO
interface FooId extends String {
  _fooIdBrand: string; // 防止类型错误
}
let fooId: FooId
fooId = 'foo' as any;
```

在需要的时候使用类型断言



# 状态函数

```ts
const { called } = new class {
  count = 0;
  called = () => {
    this.count++;
    console.log(`Called : ${this.count}`);
  };
}();

called(); // Called : 1
called(); // Called : 2
```

模拟static

> 没啥用, 直接使用closure



# Bind 是有害的

在函数上调用 `bind` 会导致你在原始函数调用签名上将会完全失去类型的安全检查

使用类型注解的箭头函数

## 类成员

使用箭头函数



# 柯里化

仅仅需要使用一系列箭头函数



# 泛型的实例化类型

为一个特定的类型创建单独的版本

```ts
class Foo<T> {
  foo: T;
}
const FooNumber = Foo as { new (): Foo<number> }; // ref 1
type FooNumberType = Foo<number>
let foo:FooNumberType = new FooNumber()
```

类型断言模式是不安全的

## 继承

```ts
class FooNumber extends Foo<number> {}
```



# 对象字面量的惰性初始化

## 最好的解决方案

最*好*的解决方案就是在为变量赋值的同时，添加属性及其对应的值

## 快速解决方案

```ts
let foo = {} as any;
```

## 折中的解决方案

interface

```ts
interface Foo {
  bar: number;
  bas: string;
}

let foo = {} as Foo;
```



# 类是有用的

> 不要再使用模块模式（利用 JavaScript 的闭包）

使用类不仅仅有利于开发者，在创建基于类的更出色可视化工具中



# `export default` 被认为是有害的

可维护性的问题：

- 不利于重构
- 有额外导出必须兼顾导入语法



## 可发现性差

不利于获取智能提示

## 自动完成

```ts
import { /* here */ } from './foo'
```



## CommonJS 互用

```ts
const { default } = require('module/foo') // bad
const { Foo } = require('module/foo') // good
```

## 容易拼写错误

## 再次导出

再次导出是没必要的，但是在 `npm` 包的根文件 `index` 却是很常见

```ts
// bad
import Foo from './foo'；
export { Foo } 
// good
export * from './foo' 
```

## 动态导入

```ts
// bad
const HighChart = await import('https://code.highcharts.com/js/es-modules/masters/highcharts.src.js');
HighChart.default.chart('container', { ... }); // Notice `.default`
// good                      
const { HighChart } = await import('https://code.highcharts.com/js/es-modules/masters/highcharts.src.js');                      
```



# 减少 setter 属性的使用

倾向于使用更精确的 `set/get` 函数（如 `setBar`, `getBar`）

> observer模式除外



# 创建数组

可以在创建数组时使用 ES6 的 `Array.prototype.fill` 方法为数组填充数据



#  谨慎使用 `--outFile`

推荐你使用外部模块，让构建工具创建单文件的 `.js`



# TypeScript 中的静态构造函数

```ts
class MyClass {
  static initalize() {
    //
  }
}

MyClass.initalize();
```

# 单例模式

如果你不想延迟初始化，你可以使用 `namespace` 替代

对大部分使用者来说，`namespace` 可以用模块来替代。

> WARNING
>
> 单例只是[全局](https://stackoverflow.com/questions/137975/what-is-so-bad-about-singletons/142450#142450)的一个别称。

# 函数参数

如果你有一个含有很多参数或者相同类型参数的函数，那么你可能需要考虑将函数改为接收对象的形式

> 如果你的函数足够简单，并且你不希望增加代码，忽略这个建议。



# Truthy

## 明确的

操作符 `!!`



# 构建切换

`process.env.NODE_ENV`



# 类型安全的 Event Emitter

为每个事件类型创建一个 emitter

```ts
const onFoo = new TypedEvent<Foo>();
const onBar = new TypedEvent<Bar>();
// Emit:
onFoo.emit(foo);
onBar.emit(bar);

// Listen:
onFoo.on(foo => console.log(foo));
onBar.on(bar => console.log(bar));

export interface Listener<T> {
  (event: T): any;
}

export class TypedEvent<T> {
  private listeners: Listener<T>[] = [];
  private listenersOncer: Listener<T>[] = [];

  public on = (listener: Listener<T>) => {
    this.listeners.push(listener)
  };
}

```

- 事件的类型，能以变量的形式被发现。
- Event Emitter 非常容易被重构。
- 事件数据结构是类型安全的。



## infer

需要熟悉infer、extends并配合三元表达式推倒类型



# TypeScript 编译原理

```text
源码 ~~扫描器~~> Tokens ~~解析器~~> AST ~~发射器~~> JavaScript
```



# **TypeScript FAQs**

# 并不是 bug

- 两个空的类，可以彼此代替
- 可以在一个返回值为 void 的函数中使用一个返回值不为 `void` 的函数
- 可以使用一个更短的参数列表，而不是一个期望的长参数列表
- 类的 `private` 成员，在运行时实际上是可见



# 类型系统的行为

**结构化类型**

结构化类型系统背后的思想是如果他们的成员类型是兼容的，则他们是兼容的



# 类

不要创建空类

## 名义上的类

当一个成员是 `private` 或者 `protected` 时，它们必须来自同一个声明，才能被视为与另一个 `private` 或者 `protected` 的成员相同。



 `typeof MyClass` 是指 `MyClass` 的类型



子类属性初始值设定会覆盖基类构造函数中设置的值

> js设计缺陷，使用多态回避此问题



避免接口继承类



## 不扩展 Error、Array、Map 内置函数

- 通过这些子类的构造函数返回的对象中，方法可能是 `undefined`
- `instanceof` 将会在子类的实例和自身实例中被中断



手动调整原型

```ts
class FooError extends Error {
  constructor(m: string) {
    super(m);

    // Set the prototype explicitly.
    Object.setPrototypeOf(this, FooError.prototype);
  }

  sayHello() {
    return 'hello ' + this.message;
  }
}
```

> 不要使用！



#  泛型

绝不应该有未使用类型的参数。该类型会有无法预料的兼容性，同时在函数调用中也无法获取正确的泛型类型接口



## 不要在泛型函数中写 `typeof T`、`new T`, 或者 `instanceof T`

```typescript
function create<T>(ctor: { new(): T }) {
  return new ctor();
}
var c = create(MyClass); // c: MyClass

function isReallyInstanceOf<T>(ctor: { new(...args: any[]): T }, obj: T) {
  return obj instanceof ctor;
}
```



# 类型守卫

类型缩小的前提是类型兼容

如果T不兼容Foo则没有缩小到Foo的必要



# JSX 和 React

区分类的实例与静态类



#  命令行的行为

必要时使用三斜线指令引入声明文件



## `Exported variable [name] has or is using private name [name]`

- 导出相关类型中使用的声明
- 当编写声明的时候，显示的为编译器指定类型注解



为了确保添加一个新文件时，输出不会被修改，你应该在命令行中或 `tsconfig.json` 指定一个 `--rootDir`。



# `tsconfig.json` 的行为

## exclude

如果你想从编译中排除一个文件，你需要排除所有具有 `import` 或者 `<reference path="...">` 指令的文件。

使用 `tsc --listFiles` 来列出在编译时包含了哪些文件，`tsc --traceResolution` 来看看它们为什么会被包含在编译中。



## `include`

1. 使用 `files` 列表，
2. 在目录中添加 `///<reference path="">` 指令



## 当我使用 JavaScript 文件时，为什么我会得到 `error TS5055: Cannot write file 'xxx.js' because it would overwrite input file` 错误

- 不想编译 JavaScript 文件:将 `allowJs` 选项设置为 `false`；
- 想要包含和编译这些 JavaScript 文件, 设置 `outDir` 或者 `outFile` 选项，定向到其他位置
- 仅仅是想包含这些 JavaScript 文件，但是不需要编译，设置 `noEmit` 选项为 `true` 可以跳过编译检查


