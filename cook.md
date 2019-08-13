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



## 3.接口

**这个是重点**

使用duck type进行契约编程

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
```

> 这里不在重复声明函数的类型，因为在接口已经声明了，所以函数类型的声明尽量靠近函数定义