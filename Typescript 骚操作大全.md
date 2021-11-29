## 一、排除原类型中特定 Key 或者特定 Value

当你要 **排除(Omit)** 或者 **选取(Pick)** 某些属性的时候，当然你可以通过 `Omit<IOrigin, "key1" | "key2">` 一个个进行处理，但是有没有一种方法可以 **批量处理** 呢，下面就给你演示骚操作

```typescript
interface IOrigin {
  [name: number]: string;
  name: string;
  gender: string;
  age: number;
  getName: () => string;
  getGener: () => string;
  getAge: () => number;
}

/**
 * 简单 Omit 排除特定 Key 的属性
 * {
 *    [name: number]: string;
 *    gender: string;
 *    age: number;
 *    getGener: () => string;
 *    getAge: () => number;
 * }
 */
type OmitSimply = Omit<IOrigin, "name" | "getName">;

// 通过 infer 排除特定 Key 的属性
type OmitInferKey<T, R> = {
  [K in keyof T as K extends R ? never : K]: T[K];
};

/**
 * 排除 Key 类型为 number 的属性
 * {
 *    name: string;
 *    gender: string;
 *    age: number;
 *    getName: () => string;
 *    getGener: () => string;
 *    getAge: () => number;
 * }
 */
type OmitInferKeyNumber = OmitInferKey<IOrigin, number>;

/**
 * 排除 Key 类型为 `get${string}` 的属性
 * {
 *    [name: number]: string;
 *    name: string;
 *    gender: string;
 *    age: number;
 * }
 */
type OmitInferKeyRegExp = OmitInferKey<IOrigin, `get${string}`>;

// 通过 infer 排除特定 Value 的属性
type OmitInferValue<T, R> = {
  [K in keyof T as T[K] extends R ? never : K]: T[K];
};

/**
 * 排除 Value 类型为 Function 的属性
 * {
 *    [name: number]: string;
 *    name: string;
 *    gender: string;
 *    age: number;
 * }
 */
type OmitInferKeyFunction = OmitInferValue<IOrigin, Function>;
```

> 这里演示下“排除(Omit)”如何实现，你可以举一反三，实现“选取(Pick)”

## 二、类型中某些属性只能“二选一”

当我们不单单要明确定义参数的类型，而且如果参数为  `object`  的话，还有可能出现  `object`  里面某两个属性是冲突，只能“二选一”的情况。

```typescript
interface IMyParams {
  a: number;
  b: number;
}

// 如果我们需要将一个参数定义为对象，并且其属性 a 或者 b 必须要传递一个的话
function calc(params: IMyParams): number {
  if (params.a) {
    console.log(params.a + 1);
  } else {
    console.log(params.b + 1);
  }
}
```

### 定义了一个 `EitherOr` 的类型

```typescript
type FilterOptional<T> = Pick<
  T,
  Exclude<
    {
      [K in keyof T]: T extends Record<K, T[K]> ? K : never;
    }[keyof T],
    undefined
  >
>;

type FilterNotOptional<T> = Pick<
  T,
  Exclude<
    {
      [K in keyof T]: T extends Record<K, T[K]> ? never : K;
    }[keyof T],
    undefined
  >
>;

type PartialEither<T, K extends keyof any> = {
  [P in Exclude<keyof FilterOptional<T>, K>]-?: T[P];
} & { [P in Exclude<keyof FilterNotOptional<T>, K>]?: T[P] } & {
  [P in Extract<keyof T, K>]?: undefined;
};

type Object = {
  [name: string]: any;
};

export type EitherOr<O extends Object, L extends string, R extends string> = (
  | PartialEither<Pick<O, L | R>, L>
  | PartialEither<Pick<O, L | R>, R>
) &
  Omit<O, L | R>;
```

### 使用例子：

```typescript
// a、b二选一，并且必须传递一个
type RequireOne = EitherOr<
  {
    a: number;
    b: string;
  },
  "a",
  "b"
>;

// a、b二选一，或者都不传
type RequireOneOrEmpty = EitherOr<
  {
    a?: number;
    b?: string;
  },
  "a",
  "b"
>;
```

### 实际应用：

```tsx
interface IColumn {
  title: string;
  dataIndex: string;
  render: () => React.ReactNode;
}
// 熟悉 antd 的同学应该都知道，如果传递了 render 的话，其他 dataIndex 其实就没意义
// 换个角度来说，其实它们两个是“二选一”的属性
interface ITableProps {
  columns: Array<EitherOr<IColumn, "dataIndex", "render">>;
}
function Table(props: ITableProps) {
  // TODO
}
```

## 三、在调用方法时，判断其中一个参数的类型决定其他参数类型甚至返回值类型

先来的平时写的简单函数

```typescript
/**
 * 正常写法
 * 对所有参数都进行“确定”的定义
 */
function simply(
  input: number | string,
  callback: (result: number | string) => void
) {
  callback(input);
}
simply(2, function (result) {
  /**
   * 此时 ts 会提示你 result 为 string|number
   * 那很明显就不对嘛，第一个参数都输入 number，但是它有点蠢，并不懂
   */
  console.log(result);
});
```

通过给函数增加**输入定义**来明确

`function fn<T>(){}`

```typescript
function definition<T>(input: T, callback: (result: T) => void) {
  callback(input);
}

definition(2, function (result) {
  console.log(result); // 提示为 number
});

definition("i'm sb", function (result) {
  console.log(result); // 提示为 string
});
```

来个复杂点的例子

```typescript
interface IData {
  name: string;
  age: number;
}

interface IComplexOption<N, A> {
  showName?: N;
  showAge?: A;
  callback: (
    result: Pick<
      IData,
      /**
       * 如果 N 为 true 时会 Pick 到 name
       * 如果 A 为 true 时会 Pick 到 age
       */
      (N extends true ? "name" : never) | (A extends true ? "age" : never)
    >
  ) => void;
}

/**
 * 当 showName 为 true 时，callback 的 result 中才会有 name
 * 当 showAge 为 true 时，callback 的 result 中才会有 age
 */
function complex<N extends boolean = false, A extends boolean = false>(
  data: IData,
  option: IComplexOption<N, A>
) {
  const result: any = {};

  option.showName && (result.name = data.name);
  option.showAge && (result.age = data.age);

  option.callback(result);
}

complex(
  { name: "tellyourmad", age: 11 },
  {
    /**
     * showName 和 showAge 都为 true 时
     * callback 中的 result 会包含 name 和 age
     */
    showName: true,
    showAge: true,
    callback: function (result) {
      console.log(result.name);
      console.log(result.age);
    },
  }
);

complex(
  { name: "tellyourmad", age: 11 },
  {
    showName: true,
    showAge: false,
    callback: function (result) {
      console.log(result.name); // 不会报错
      console.log(result.age); // 报错，提示没有 age
    },
  }
);

complex(
  { name: "tellyourmad", age: 11 },
  {
    callback: function (result) {
      console.log(result.name); // 报错，提示没有 name
      console.log(result.age); // 报错，提示没有 age
    },
  }
);
```

看到这里你大概率是有点懵的，其实关键点只有一个，就是通过

```typescript
function fn<T>(
    input: T,
    cb: (result: T) => void
){
    ...
}
```

但是注意的是，这里如果单纯定义`T`的话，其实有两个问题没解决

- 定义可选参数，要通过 `=` 符号来明确
- 参数类型定义，要通过 `extends` 关键字来明确

```typescript
function complex<
  N extends boolean = false, // N 类型为 boolean，默认值为 false
  A extends boolean = false
>(
  data: IData,
  option: IComplexOption<N, A>
) {
  ...
  ...
}
```

p.s. 其实类似的解决办法就是 **重载**，譬如：

```typescript
function add(a: number, b: number): number;
function add(a: string, b: string): string;
function add(a: string, b: number): string;
function add(a: number, b: string): string;
function add(a: any, b: any) {
  if (typeof a === "string") {
    return a + b + "!";
  } else {
    return a + b;
  }
}
```

但是这种方法需要对每一个情况都要定义（譬如上面就定义了四种情况）

## 四、真正的枚举

typescript 中的枚举只是纯粹的 key-value，并不像 java 那样还有 toString 方法来获取 label

```typescript
enum STATUS {
  DEFAULT = "default",
  PENDING = "pending",
  SUCCESS = "success",
  FAIL = "FAIL",
}
```

但是很多时候我们需要的是 key-value-label，譬如我们用一个 antd 的 Select 组件需要如下格式：

p.s. 当然有的人会直接声明个数组就行了，但是咱们这是用了 ts，必须得给自己加戏哎～

```typescript
[
  {
    key: "DEFAULT",
    label: "默认",
    value: "default",
  },
  ...
]
```

下面用演示下如何用两个 enum 来实现

```typescript
// 再弄一个枚举，用来存 label
enum STATUS_MAP {
  DEFAULT = "默认",
  PENDING = "待定",
  SUCCESS = "成功",
  FAIL = "失败",
}

// 封装一个 enumEntries 方法，用来转化
type Enum = {
  [name in string | number]: string | number;
};
type SelectOption = {
  key: string;
  value: string | number;
  label: string | number;
};
function enumEntries(source: Enum, map?: Enum): Array<SelectOption> {
  return Object.keys(source)
    .filter((item) => isNaN(+item))
    .map((item) => ({
      key: item,
      value: source[item],
      label: map && typeof map[item] !== "undefined" ? map[item] : source[item],
    }));
}

// 调用下就能出结果啦
console.log(enumEntries(STATUS, STATUS_MAP));
```

反过来怎么根据枚举中某个 key 反响推倒出 label 呢，这里再封装一个方法：

```typescript
function enumToString(type: any, source: Enum, map?: Enum): string {
  let sourceStr: any = Object.entries(source).find((item) => item[1] === type);
  sourceStr = typeof sourceStr === "undefined" ? "" : sourceStr[0];
  const mapStr = map ? map[sourceStr] : "";
  return (mapStr || sourceStr).toString();
}

console.log(
  enumToString(
    STATUS.PENDING, // 枚举中某个 key
    STATUS, // key-value 枚举
    STATUS_MAP // key-label 枚举
  )
);
// 输出 "待定"
```

总结一下

- 定义 key-value
- 定义 key-label
- 封装 enumEntries 方法，生成 key-value-label
- 封装 enumToString 方法，输入 value 返回 label
