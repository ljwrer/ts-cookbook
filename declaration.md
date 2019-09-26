# 声明文件

## 1. 结构

### 识别库的类型

#### 全局库

- global.d.ts

#### 模块化库

*UMD库模板*

- module.d.ts
- module-class.d.ts
- module-function.d.ts

*模块插件或UMD插件*

- module-plugin.d.ts

#### 全局插件

如扩展prototype

- global-plugin.d.ts

#### 全局修改的模块

如模块中扩展prototype

- global-modifying-module.d.ts



### 使用依赖

#### 依赖全局库

```typescript
/// <reference types="someLib" />
```

#### 依赖模块

```typescript
import * as moment from "moment";
```

#### 依赖UMD库

*从全局库*

```typescript
/// <reference types="moment" />
```

*从一个模块或UMD库*

```typescript
import * as someLib from 'someLib';
```



### 补充说明

#### 防止命名冲突

不建议在全局作用域定义类型

使用库定义的全局变量名来声明命名空间类型

```typescript
declare namespace cats {
    interface KittySettings { }
}
```

#### ES6模块插件的影响

ES模块被当作是不可改变

#### ES6模块调用签名的影响

顶层的模块对象 *永远不能*被调用。 十分常见的解决方法是定义一个 `default`导出到一个可调用的/可构造的对象； 一会模块加载器助手工具能够自己探测到这种情况并且使用 `default`导出来替换顶层对象



---



## 2. 举例

### 例子

#### 全局变量

```typescript
/** 组件总数 */
declare var foo: number;
declare const bar: string;
declare let baz: boolean;
```

> 可以写注释

#### 全局函数

```typescript
declare function greet(greeting: string): void;
```

#### 带属性的对象

```typescript
declare namespace myLib {
    function makeGreeting(s: string): string;
    let numberOfGreetings: number;
}
```

#### 函数重载

```typescript
declare function getWidget(n: number): Widget;
declare function getWidget(s: string): Widget[];
```

#### 可重用类型（接口）

```typescript
interface GreetingSettings {
  greeting: string;
  duration?: number;
  color?: string;
}

declare function greet(setting: GreetingSettings): void;
```

#### 可重用类型（类型别名） 

```typescript
type GreetingLike = string | (() => string) | MyGreeter;

declare function greet(g: GreetingLike): void;
```

#### 组织类型

使用命名空间组织类型

```typescript
declare namespace GreetingLib {
    interface LogOptions {
        verbose?: boolean;
    }
    interface AlertOptions {
        modal: boolean;
        title?: string;
        color?: string;
    }
}
```

在一个声明中创建嵌套的命名空间

```typescript
declare namespace GreetingLib.Options {
    // Refer to via GreetingLib.Options.Log
    interface Log {
        verbose?: boolean;
    }
    interface Alert {
        modal: boolean;
        title?: string;
        color?: string;
    }
}
```

#### 类

```typescript
declare class Greeter {
    constructor(greeting: string);

    greeting: string;
    showGreeting(): void;
}
```



---



## 3. 规范

### 普通类型

#### `Number`，`String`，`Boolean`和`Object`

*不要*使用如下类型`Number`，`String`，`Boolean`或`Object`。 这些类型指的是非原始的装盒对象

*应该*使用类型`number`，`string`，and `boolean`

使用非原始的`object`类型来代替`Object`

#### 范型

*不要*定义一个从来没使用过其类型参数的泛型类型

### 回调函数类型

#### 回调函数返回值类型

*不要*为返回值被忽略的回调函数设置一个`any`类型的返回值类型

*应该*给返回值被忽略的回调函数设置`void`类型的返回值类型

> 使用`void`相对安全，因为它防止了你不小心使用返回值

#### 回调函数里的可选参数

*不要*在回调函数里使用可选参数除非你真的要这么做

> 总是允许提供一个接收较少参数的回调函数
>
> 参考协变与逆变，函数类型兼容问题

#### 重载与回调函数

*不要*因为回调函数参数个数不同而写不同的重载

*应该*只使用最大参数个数写一个重载

> 同上,参数少的回调函数首先允许错误类型的函数被传入

### 函数重载

*不要*把一般的重载放在精确的重载前面

*应该*排序重载令精确的排在一般的之前

> 按声明的顺序解析

#### 使用可选参数

*不要*为仅在末尾参数不同时写不同的重载

*应该*尽可能使用可选参数(返回类型一致)

> TypeScript解析签名兼容性时会查看是否某个目标签名能够使用源的参数调用， *且允许外来参数*,  此时重载兼容性更好（只要有匹配的重载即可）
>
> strict null checking开启时，可选参数兼容undefined

#### 使用联合类型

*不要*为仅在某个位置上的参数类型不同的情况下定义重载

 *应该*尽可能地使用联合类型

> 便于参数透传



---



## 4. 深入

了解如果书写复杂的暴露出友好API的声明文件

#### 核心概念

*类型*

- 类型别名声明（`type sn = number | string;`）
- 接口声明（`interface I { x: number[]; }`）
- 类声明（`class C { }`）
- 枚举声明（`enum E { A, B, C }`）
- 指向某个类型的`import`声明

*值*

- `let`，`const`，和`var`声明
- 包含值的`namespace`或`module`声明
- `enum`声明
- `class`声明
- 指向值的`import`声明
- `function`声明

*命名空间*

类型可以存在于*命名空间*里

#### 简单的组合：一个名字，多种意义

同一个标识符可以同时表示一个类型，一个值或一个命名空间，如何解析取决于上下文

*内置组合*

`class C { }`声明创建了两个东西： *类型*`C`指向类的实例结构， *值*`C`指向类构造函数。 

枚举声明拥有相似的行为

*用户组合*

可同时做为类型和值



#### 高级组合

##### *利用`interface`添加*

使用一个`interface`往别一个`interface`声明或类里添加额外成员

```typescript
interface Foo {
  x: number;
}
// ... elsewhere ...
interface Foo {
  y: number;
}
/* -------- */
class Bar {
  x: number;
}
// ... elsewhere ...
interface Bar {
  y: number;
}
```

##### *使用`namespace`添加*

`namespace`声明可以用来添加新类型，值和命名空间

```typescript
// 添加静态成员到一个类
class C {
}
// ... elsewhere ...
namespace C {
  export let x: number;
}
```

```typescript
// 给类添加一个命名空间类型
class C {
}
// ... elsewhere ...
namespace C {
  export interface D { }
}
```

```typescript
// 合并namespace声明
namespace X {
  export interface Y { }
  export class Z { }
}

// ... elsewhere ...
namespace X {
  export var Y: number;
  export namespace Z {
    export class C { }
  }
}
type X = string;
```

#### 使用`export =`或`import`

`export`和`import`声明会导出或导入目标的*所有含义*



---



## 5. 发布

1. 与你的npm包捆绑在一起，或
2. 发布到npm上的[@types organization](https://www.npmjs.com/~types)。

### npm

*package.json*

```json
{
    "name": "awesome",
    "author": "Vandelay Industries",
    "version": "1.0.0",
    "main": "./lib/main.js",
    "types": "./lib/main.d.ts"
}
```

#### 依赖

`dependencies`

#### 危险信号

*不要*在声明文件里使用`/// <reference path="..." />`

*应该*使用`/// <reference types="..." />`代替

#### 打包所依赖的声明

- *不要*把依赖的包放进你的包里，保持它们在各自的文件里。
- *不要*将声明拷贝到你的包里。
- *应该*依赖于npm类型声明包，如果依赖包没包含它自己的声明的话。

#### 发布到[@types](https://www.npmjs.com/~types)

[types-publisher工具](https://github.com/Microsoft/types-publisher)



## 6. 使用

```bash
npm install --save @types/moduleName
```

