# 项目配置

### tsconfig.json

通常来讲，不推荐只有扩展名的不同来区分同目录下的文件

#### 指定编译文件

- files
- include
- exclude

#### 指定类型文件

- compilerOptions
  - typeRoots
  - types

#### 扩展配置文件

- extends



### 编译选项

> `/!*`开头表示版权信息

#### 开启所有严格检查

- strict



### 项目引用

> 保持清晰的模块依赖，全量构建未尝不可

#### 工程引用

它是一个对象的数组，指明要引用的工程，实际加载的是它*输出*的声明文件

```json
{
  "references": [
    { "path": "../src" }
  ]
}
```

#### declarationMaps`

在某些编辑器上，你可以使用诸如“Go to Definition”，重命名以及跨工程编辑文件等编辑器特性