# Initialization

---

## 1. Overview

---

Initialization 是 object 在定义时获得初始值的过程。

最直接的例子是：

```c
int count = 10;
```

这里：

- `count` 是一个 `int` object；
- `= 10` 是 initializer；
- object 获得初始值 `10` 的过程就是 initialization。

初始化并不只发生在整数上：

```c
float voltage = 3.3f;
bool ready = true;
char grade = 'A';
```

理解 initialization 时，需要掌握三件事：

1. 有 initializer 时，初始值怎样提供；
2. 没有 initializer 时，object 会得到什么值；
3. initialization 与 object 创建之后的 assignment 有什么不同。

---

## 2. Core Concepts

---

### 2.1 Explicit Initialization

在定义 object 时写出 initializer，称为显式初始化。

基本形式是：

```c
type identifier = initializer;
```

例如：

```c
int count = 10;
float voltage = 3.3f;
char grade = 'A';
```

Initializer 也可以是 expression：

```c
int base = 10;
int total = base + 5;
```

这里 `base + 5` 的结果用于初始化 `total`。

对于 scalar type，也可以使用花括号：

```c
int count = {10};
```

它仍然是在初始化同一个 `int` object。

---

### 2.2 Initialization and Assignment

Initialization 发生在 object 获得初始值时。

Assignment 发生在 object 已经存在之后，用新值替换它当前保存的值。

```c
int count = 10;  /* initialization */

count = 20;      /* assignment */
```

第一行定义并初始化 `count`。

第二行没有创建新的 object，而是修改已经存在的 `count`。

因此：

```c
int value;
value = 5;
```

这里 `value = 5;` 是 assignment，不是 initializer。

---

### 2.3 Initialization Without an Explicit Initializer

没有显式 initializer 时，结果取决于 object 的 storage duration。

#### Automatic Storage Duration

普通 block-scope local object 通常具有 automatic storage duration。

```c
void function(void)
{
    int count;
}
```

这里 `count` 没有显式 initializer，它的 representation 是 indeterminate。

因此它不能被理解为自动获得 `0`。

#### Static Storage Duration

具有 static storage duration 的 object 如果没有显式 initializer，会接受默认初始化。

例如：

```c
int global_count;

void function(void)
{
    static int call_count;
}
```

`global_count` 和 `call_count` 都具有 static storage duration，因此它们的初始整数值都是 `0`。

对于常见类型，默认初始化的结果包括：

| Type category | Initial value |
| --- | --- |
| integer | `0` |
| floating | positive zero |
| pointer | null pointer |
| aggregate | 各 element / member 递归地按相同规则初始化 |

Storage duration 的完整规则见 [007-storage-duration-and-lifetime](../007-storage-duration-and-lifetime/)。

---

### 2.4 Brace-Initialized Aggregates

Array、structure 和 union 这类由多个部分组成的 object 可以使用 brace-enclosed initializer list。

以 array 为例：

```c
int values[4] = {10, 20, 30, 40};
```

Initializer list 中的值依次初始化 array elements。

如果只提供部分 initializers：

```c
int values[4] = {10, 20};
```

结果是：

| Element | Initial value |
| --- | ---: |
| `values[0]` | `10` |
| `values[1]` | `20` |
| `values[2]` | `0` |
| `values[3]` | `0` |

没有显式提供 initializer 的剩余 elements 会按默认初始化规则初始化。

因此下面的写法会把整个 integer array 初始化为零：

```c
int values[4] = {0};
```

Structure 也使用相同的 initializer-list 思路：

```c
struct Point
{
    int x;
    int y;
};

struct Point p = {10, 20};
```

这里 `10` 初始化 `p.x`，`20` 初始化 `p.y`。

Array 和 structure 本身的完整知识分别属于 [05-arrays-strings-and-buffers](../../05-arrays-strings-and-buffers/) 和 [07-objects-and-data-layout](../../07-objects-and-data-layout/)。

---

### 2.5 Designated Initializers

C 可以在 initializer list 中明确指定要初始化的 element 或 member。

Array designator 使用 `[index] =`：

```c
int values[5] =
{
    [1] = 10,
    [4] = 20
};
```

这会得到：

| Element | Initial value |
| --- | ---: |
| `values[0]` | `0` |
| `values[1]` | `10` |
| `values[2]` | `0` |
| `values[3]` | `0` |
| `values[4]` | `20` |

Structure designator 使用 `.member =`：

```c
struct Point
{
    int x;
    int y;
};

struct Point p =
{
    .y = 20,
    .x = 10
};
```

Designated initializer 让 initializer 明确对应到指定的 element 或 member，而不必完全依赖声明顺序。

---

### 2.6 Character Array Initialization

Character array 可以直接由 string literal 初始化：

```c
char text[] = "hello";
```

这会创建足够大的 array，并把字符串中的字符以及结尾的 null character `'\0'` 放入 array。

等价的基本形式可以写成：

```c
char text[] = {'h', 'e', 'l', 'l', 'o', '\0'};
```

字符串和 character array 的完整规则见 [05-arrays-strings-and-buffers](../../05-arrays-strings-and-buffers/)。

---

### 2.7 Initialization and Storage Duration

Storage duration 还会影响 initializer 可以使用什么 expression。

Automatic object 的 initializer 可以使用运行时计算得到的值：

```c
int read_sensor(void);

void function(void)
{
    int value = read_sensor();
}
```

这里 `read_sensor()` 在函数执行期间产生用于初始化 `value` 的值。

具有 static storage duration 的 object，其 initializer 必须满足静态初始化所要求的 constant-expression / string-literal 规则。

例如：

```c
static int limit = 100;
```

这里 `100` 是可以用于 static object initialization 的 constant expression。

更完整的 storage duration 规则见 [007-storage-duration-and-lifetime](../007-storage-duration-and-lifetime/)。

---

### 2.8 Putting the Concepts Together

下面几种声明体现了不同的 initialization 情况：

```c
int global_count;

void function(void)
{
    int a = 10;
    int b;
    static int c;
    int values[4] = {1, 2};
}
```

可以这样理解：

| Object | Initialization |
| --- | --- |
| `global_count` | static storage duration，没有显式 initializer，初始值为 `0` |
| `a` | 显式初始化为 `10` |
| `b` | automatic storage duration，没有显式 initializer，representation indeterminate |
| `c` | static storage duration，没有显式 initializer，初始值为 `0` |
| `values` | 前两个 elements 为 `1`、`2`，其余 elements 为 `0` |

---

## 3. Related Knowledge

---

- [001-variables-declarations-and-definitions](../001-variables-declarations-and-definitions/) — declaration, definition, initializer, and object
- [002-fundamental-types](../002-fundamental-types/) — scalar types used in initialization examples
- [007-storage-duration-and-lifetime](../007-storage-duration-and-lifetime/) — automatic and static storage duration
- [05-arrays-strings-and-buffers](../../05-arrays-strings-and-buffers/) — array and string semantics
- [07-objects-and-data-layout](../../07-objects-and-data-layout/) — structures, unions, and object layout

---

## 4. References

---

1. **ISO/IEC 9899:2024 (C23), 6.7.11 Initialization** — initializer syntax and initialization semantics.
2. **WG14 N3220 — ISO/IEC 9899:2024 working draft, 6.7.11 Initialization**  
   https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3220.pdf
3. **cppreference — Initialization**  
   https://en.cppreference.com/c/language/initialization
4. **cppreference — Scalar initialization**  
   https://en.cppreference.com/c/language/scalar_initialization
5. **cppreference — Array initialization**  
   https://en.cppreference.com/c/language/array_initialization
6. **cppreference — Struct and union initialization**  
   https://en.cppreference.com/c/language/struct_initialization
7. **cppreference — Storage-class specifiers**  
   https://en.cppreference.com/c/language/storage_duration
