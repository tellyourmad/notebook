在 TypeScript 中有一些比较重要的关键字

1. `typeof`
2. `keyof`
3. `in`
4. `extends`
5. `as`
6. `infer`

它们在实际使用中往往不会单独出现，譬如使用 `keyof` 的场景中往往也需要用到 `typeof`，而使用 `in` 的场景也往往需要用到 `keyof` 和 `typeof`。

所以这里的排序同时也代表着学习的顺序～

## `typeof`

TS 和 JS 里面都有 `typeof` 关键字，并且其二者的作用都差不多

**负责转化 JS data（常量或者变量）**

差异是转化结果不一样：

- 在 TS 中，是转化成 **一个 TS 类型定义，也就是`type`**，JS data -> TS type
- 在 JS 中，是转化成 **一个字符串，表示未经计算的操作数的类型**，JS data -> JS data

而使用的地方决定了它是 TS 还是 JS 的 `typeof`

### 基本用法

```typescript
// JS 对象
const JPeople = {
  name: "张大炮",
  age: 18,
};

// TS 类型 type
type TPeople = {
  name: string;
  age: number;
};

// TS 类型 interface
interface IPeople {
  name: string;
  age: number;
}

// 以下方式会被认为是 JS 的 typeof
const JsKeyword1 = typeof JPeople; // const JsKeyword1 = "string"
const JsKeyword2 = typeof TPeople; // 错误，TPeople 不是数据，不能获取数据类型
const JsKeyword3 = typeof IPeople; // 错误，IPeople 不是数据，不能获取数据类型

// 以下方式会被认为是 TS 的 typeof
type TsType1 = typeof JPeople; // type TsType1 = {name: string; age: number}
type TsType2 = typeof TPeople; // 错误，TPeople 已经是 TS 类型了，不能再转化
type TsType3 = typeof IPeople; // 错误，IPeople 已经是 TS 类型了，不能再转化
```

在 TS 中，`typeof` 只能对**数据**进行转化，所以不能转化 `type` 和 `interface`

### 枚举 Enum

`typeof` 可以转化 `enum`，因为其本质是 **JS 对象**

```typescript
// 纯数字枚举
enum ENUM_NUMBER {
  FIRST,
  SECOND,
  THIRD,
}

// 非纯数字枚举
enum ENUM_WHEREVER {
  FIRST = "第一个",
  SECOND = "第二个",
  THIRD = 333,
}
```

TS 中定义枚举 `enum` 后，在 JS 中会被解析成对象，如下所示：

```javascript
var ENUM_NUMBER;
(function (ENUM_NUMBER) {
  ENUM_NUMBER[(ENUM_NUMBER["FIRST"] = 0)] = "FIRST";
  ENUM_NUMBER[(ENUM_NUMBER["SECOND"] = 1)] = "SECOND";
  ENUM_NUMBER[(ENUM_NUMBER["THIRD"] = 2)] = "THIRD";
})(ENUM_NUMBER || (ENUM_NUMBER = {})); // 字符串枚举

(function (ENUM_WHEREVER) {
  ENUM_WHEREVER["FIRST"] = "\u7B2C\u4E00\u4E2A";
  ENUM_WHEREVER["SECOND"] = "\u7B2C\u4E8C\u4E2A";
  ENUM_WHEREVER[(ENUM_WHEREVER["THIRD"] = 333)] = "THIRD";
})(ENUM_WHEREVER || (ENUM_WHEREVER = {}));
```

所以你可以把 `enum` 理解为 `object`，所以它是一份**数据**，那当然可以使用 `typeof` 对其进行转化

```typescript
// type TsType4 = {FIRST: number; SECOND: number, THIRD: number}
type TsTypeEnumNumber = typeof ENUM_NUMBER;

// type TsTypeEnumWherever = {FIRST: string; SECOND: string, THIRD: number}
type TsTypeEnumWherever = typeof ENUM_WHEREVER;
```

## `keyof`

`keyof` 的作用将一个 **类型** 映射为它 **所有成员名称的联合类型**

- `type` -> `type（联合类型）`
- `interface` -> `type（联合类型）`

### 基本用法

```typescript
// 用 TS interface 描述对象
interface TsInterfaceObject {
  first: string;
  second: number;
}

// 用 TS type 描述对象
type TsTypeObject = {
  first: string;
  second: number;
};

// 用 Ts type 描述基本类型别名
type TsTypeAlias = string;

class JsClass {
  private priData: number;
  private priFunc() {}

  public pubData: number;
  public pubFunc1() {}
  public pubFunc2() {}
}

/**
 * 将所描述对象中的所有 key 组合成一个联合类型
 * type NameUnionOfInterface = "first" | "second"
 */
type NameUnionOfInterface = keyof TsInterfaceObject;

/**
 * 将所描述对象中的所有 key 组合成一个联合类型
 * type NameUnionOfType = "first" | "second"
 */
type NameUnionOfTypeObj = keyof TsTypeObject;

/**
 * 对一个非对象类型使用 keyof 后，会返回其 prototype 上所有 key 组合成的一个联合类型
 * type NameUnionOfTypeAlias = number | typeof Symbol.iterator | "toString" | "charAt" | ...
 */
type NameUnionOfTypeAlias = keyof TsTypeAlias;

/**
 * 其实也是返回 prototype 上所有 key（因为其实 private 私有部分实际是放在实例化对象中，而非原型）
 * type NameUnionOfClass = "pubData" | "pubFunc1" | "pubFunc2"
 */
type NameUnionOfClass = keyof JsClass;
```

### `keyof` + `typeof`

前面提到有三个知识点

- `typeof` 可以进行 **JS data -> TS type** 的转换
- `keyof` 可以进行 **TS type -> TS (union) type** 的转换
- `enum` 其实就是 **JS (object) data**

```typescript
// JS 对象
const JsObject = {
  first: "第一个",
  second: 222,
};

// TS 枚举
enum TS_ENUM {
  FIRST,
  SECOND,
}

type NameUnionOfObject = keyof typeof JsObject; // type NameUnionOfObject = "first" | "second"
type NameUnionOfEnum = keyof typeof TS_ENUM; // type NameUnionOfObject = "FIRST" | "SECOND"
```

## `in`

关键字 `in` 是用来遍历**枚举类型**的

### 基本用法

```typescript
// TS 联合类型
type TsTypeUnion = "first" | "second";

// type TsTypeObject1 = { first: any, second: any }
type TsTypeObject1 = {
  [P in TsTypeUnion]: any;
};

// type TsTypeObject2 = { first: "first", second: "second" }
type TsTypeObject2 = {
  [P in TsTypeUnion]: P;
};
```

### `in` + `keyof`

- `keyof` 将 **描述对象的类型** 转化为 **枚举类型**
- `in` 对 **枚举类型** 进行遍历

下面用一个“脱裤子放屁”的例子来演示它们两者结合使用的场景：

```typescript
// TS 描述对象的类型
type TsTypeObject = {
  first: string;
  second: number;
};

// type TsTypeObjectNew1 = { first: any, second: any }
type TsTypeObjectNew1 = {
  [P in keyof TsTypeObject]: any;
};

// type TsTypeObjectNew2 = { first: string, second: number }
type TsTypeObjectNew2 = {
  [P in keyof TsTypeObject]: TsTypeObject[P];
};
```

### `in` + `keyof` + `typeof`

- `typeof` 将 **JS 对象** 转化为 **描述对象的类型**
- `keyof` 将 **描述对象的类型** 转化为 **枚举类型**
- `in` 对 **枚举类型** 进行遍历

处理 JS 对象：

```typescript
// JS 对象
const JsObject = {
  name: "张大炮",
  age: 18,
};

// type TsTypeObject1 = { name: any; age: any; }
type TsTypeObject1 = {
  [P in keyof typeof JsObject]: any;
};

/**
 * 等价于下面两种写法：
 * type TsTypeObject2 = { name: any; age: any; }
 * type TsTypeObject2 = typeof JsObject
 */
type TsTypeObject2 = {
  [P in keyof typeof JsObject]: typeof JsObject[P];
};
```

处理 TS 枚举：

```typescript
// 纯数字枚举
enum ENUM_NUMBER {
  FIRST,
  SECOND,
  THIRD,
}

enum ENUM_WHEREVER {
  FIRST = "第一个",
  SECOND = "第二个",
  THIRD = 333,
}

/**
 * 
    type TsType1 = {
      [x: number]: any;
      readonly FIRST: any;
      readonly SECOND: any;
      readonly THIRD: any;
    }
 */
type TsType1 = {
  [P in keyof typeof ENUM_NUMBER]: any;
};

/**
    type TsType2 = {
      [x: number]: string;
      readonly FIRST: ENUM_WHEREVER.FIRST;
      readonly SECOND: ENUM_WHEREVER.SECOND;
      readonly THIRD: ENUM_WHEREVER.THIRD;
    }
 */
type TsType2 = {
  [P in keyof typeof ENUM_WHEREVER]: typeof ENUM_WHEREVER[P];
};
```

## `extends`

在 TS 中，`extends` 大体分为三种使用场景：

- 类型继承，类型 A 去继承类型 B（`interface` 可用 `extends` 继承，`type` 不可以）
- 定义范型，约束范型必须是与目标类型“匹配的”
- 条件匹配，判断类型 A 是否“匹配”类型 B

### 类型继承

```typescript
interface IHuman {
  gender: "male" | "female";
  age: number;
}

interface IProgrammer {
  language: "java" | "php" | "javascript";
}

/**
 * interface 的继承
 * 可以同时继承多个 interface，它们之间用逗号分隔
 */
interface IWebDeveloper extends IProgrammer, IHuman {
  skill: "vue" | "react";
}

const Me: IWebDeveloper = {
  skill: "react",
  language: "javascript",
  age: 18,
  gender: "male",
};
```

### 定义范型

```typescript
enum GENDER {
  MALE,
  FEMALE,
}
// 约束范型 G 必须是 GENDER 的子类
interface IHuman<G extends GENDER> {
  gender: G;
  age: number;
}

enum LANGUAGE {
  JAVA,
  PHP,
  JAVASCRIPT,
}
// 约束范型 L 的类型并且定义默认值
interface IProgrammer<L extends LANGUAGE = LANGUAGE.JAVASCRIPT> {
  language: L;
}

interface IWebDeveloper extends IProgrammer, IHuman<GENDER.MALE> {
  skill: "vue" | "react";
}
```

### 条件匹配

可以简单理解为一个三目运算：

```typescript
// 判断范型 T 是否匹配于 number
type OnlyWantNumber<T> = T extends number ? any : never;

type Type1 = OnlyWantNumber<number>; // type Type1 = any
type Type2 = OnlyWantNumber<string>; // type Type2 = never
```

需要注意的是，如果 `extends` 左侧的类型为联合类型，会被拆解，就像数学中的分解因式子一样

`(a + b) * c => ac + bc`

```typescript
type Subtraction<T, U> = T extends U ? never : T; // 找差集
type Intersection<T, U> = T extends U ? T : never; // 找交集

type Type1 = "a" | "b" | "c";
type Type2 = "a" | "b";

/**
 * 下面三个 type 结果一样
 */
type Sub1 = Subtraction<Type1, Type2>;
type Sub2 =
  | ("a" extends Type2 ? never : "a")
  | ("b" extends Type2 ? never : "b")
  | ("c" extends Type2 ? never : "c");
type Sub3 = "c";

/**
 *
 */
type Int1 = Intersection<Type1, Type2>;
type Int2 =
  | ("a" extends Type2 ? "a" : never)
  | ("b" extends Type2 ? "b" : never)
  | ("c" extends Type2 ? "c" : never);
type Int3 = "a" | "b";
```

### `extends` + `in` + `keyof`

```typescript
type IObject = {
  a: string;
  b: number;
  c: boolean;
};

/**
 * 判断对象中 value 的类型，如果原类型是 string 则返回 string，否则返回 number
 * type TsType = { a: string; b: number; c: number; }
 */
type TsType = {
  [P in keyof IObject]: IObject[P] extends string ? string : number;
};
```

## `as`

运算符 `as` 的作用可以理解为**改变原有类型定义**，分为以下两种场景：

- 断言
- 转化

### 断言 JS 数据

```typescript
const myStr: any = "tellyourmad";
const myStrLength: number = (someValue as string).length;
```

### 转化 TS 类型

```typescript
type TsUnionType = "a" | "b" | 1 | 2;

// type TsType1 = { a: any; b: any; 1: any; 2: any; }
type TsType1 = {
  [P in TsUnionType]: any;
};

/**
 * 强制将 key 全都 as 成 number 了
 * type TsType2 = { [x: number]: any; }
 */
type TsType2 = {
  [P in TsUnionType as number]: any;
};
```

### `as` + `extends` + `in` + `keyof`

所用到的知识点：

- `keyof` 解构
- `in` 遍历
- `extends` 判断
- `as` 转换

```typescript
// 排除特定属性名
type OmitProp<T, R> = {
  [K in keyof T as K extends R ? never : K]: T[K];
  // [K in keyof T as (K extends R ? never : K)]: T[K];
};

// 排除特定属性值
type OmitValue<T, R> = {
  [K in keyof T as T[K] extends R ? never : K]: T[K];
  // [K in keyof T as (T[K] extends R ? never : K)]: T[K];
};

type IOrigin = {
  a: string;
  b: boolean;
  c: number;
  d: string;
};

/**
 * 排除属性名的类型为 "c" | "d" 的属性
 * type IWithoutProp = { a: string; b: string; }
 */
type IWithoutProp = OmitProp<IOrigin, "c" | "d">;

/**
 * 排除属性值的类型为 string | boolean 的属性
 * type IWithoutValue = { c: number; }
 */
type IWithoutValue = OmitValue<IOrigin, string | boolean>;
```

## `infer`

`infer` 用于获取 `extends` 的推导过程中出现的某个类型，最早出现于这个 [PR](https://github.com/Microsoft/TypeScript/pull/21496) 中

在 2.8 版本中，TypeScript 内置了一些与 `infer` 有关的映射类型：

- 提取函数类型的返回值类型 `ReturnType`：

```typescript
type ReturnType<T> = T extends (...args: any[]) => infer P ? P : any;

type MyFunc = () => string;

type TsType = ReturnType<MyFunc>; // type TsType = string
```

- 获取类的构造函数的参数类型 `ConstructorParameters`：

```typescript
type ConstructorParameters<T extends new (...args: any[]) => any> =
  T extends new (...args: infer P) => any ? P : never;

class MyClass {
  constructor(name: string, age: number) {}
}

type TsType = ConstructorParameters<typeof MyClass>; // type Params = [name: string, age: number]
```

- 获取实例类型 `InstanceType`：

```typescript
type InstanceType<T extends new (...args: any[]) => any> = T extends new (
  ...args: any[]
) => infer R
  ? R
  : any;

class MyClass {
  constructor(name: string, age: number) {}
}

type Instance = InstanceType<typeof MyClass>; // type Instance = MyClass
```
