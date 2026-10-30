---
name: google-cpp-code-style
description: 按 Google C++ 风格规范写、改、审 C++ 代码（命名、头文件组织、类设计、语言特性取舍、格式、注释）。用户写 C++ 新代码、review 代码、问"这么命名对不对""头文件该怎么拆""这个特性能不能用""帮我按规范整理一下"时都要主动触发，不需要用户点名。纯 C 代码、其他语言、与风格无关的报错排查不适用。
---

# Google C++ 风格规范

来源：Google C++ Style Guide 中文版（2024/04 更新，目标 C++20）。
本文是高频速查；细则见 `references/`。

## 用法

- **写新代码**：直接按下面的表落地，不用解释规则出处。
- **审已有代码**：只报真问题，按"命名 → 头文件/依赖 → 所有权与生命周期 → 接口设计 → 格式"排序。格式问题一句话带过，别刷屏 —— 那是 clang-format 的活。
- **改别人的文件**：`一致性也包括原有的一致性`。文件已有风格与本规范冲突时，跟文件走，只在新增部分用新风格。这是规范本身写明的特例，不是妥协。
- **规则冲突**：项目既有约定 > 官方 Google 规范 > 团队改编版（见 `references/variants.md`）。

## 命名速查

| 类别 | 规则 | 示例 |
|---|---|---|
| 文件 | 全小写 + 下划线，`.cc` / `.h` / `.inc` | `foo_bar.cc` ↔ `foo_bar.h` ↔ `class FooBar` |
| 类型（类、结构体、typedef、using 别名、枚举名、类型模板参数） | 大驼峰，无下划线 | `UrlTable` `PropertiesMap` `UrlTableErrors` |
| 变量、函数参数、结构体成员 | 全小写 + 下划线 | `table_name` `num_entries` |
| 类成员变量 | 同上，**尾部加下划线** | `table_name_` `pool_` |
| 常量（`constexpr`/`const` 且运行期不变，静态存储期的一律） | `k` + 大驼峰 | `kDaysInAWeek` `kMaxRetry` |
| 函数 | 大驼峰；缩写视作一个词 | `AddTableEntry()` `StartRpc()`（不是 `StartRPC`） |
| 取值/设值函数 | 与成员变量名对齐 | `int count()` / `void set_count(int count)` |
| 命名空间 | 全小写，顶层 = 项目名或团队名，不缩写 | `websearch::index` |
| 枚举值 | 优先常量风格 | `kErrorOutOfMemory`（旧码里的 `OUT_OF_MEMORY` 不用改） |
| 宏 | 全大写 + 下划线（前提是你真的躲不开宏） | `MY_MACRO_THAT_SCARES_SMALL_CHILDREN` |
| 非类型模板参数 | 按变量规则 | `template <int kBufSize>` |

通用原则：名字要有描述性，别心疼字符。`num_errors` 好过 `nerr`，`price_count_reader` 好过 `pc_reader`。公认缩写（`i` 做迭代变量、`T` 做模板参数、`DNS`、`num`）可以用；砍字母造出来的缩写（`cstmr_id`）和只有本组懂的缩写（`wgc_connections`）不行。

完整规则 + 边界情况见 `references/naming.md`。

## 头文件

```cpp
// foo/server/fooserver.h
#ifndef FOO_SERVER_FOOSERVER_H_     // <项目>_<路径>_<文件名>_H_
#define FOO_SERVER_FOOSERVER_H_
...
#endif  // FOO_SERVER_FOOSERVER_H_
```

include 顺序，每组之间空一行、组内按字母序：

```cpp
#include "foo/server/fooserver.h"   // 1. 配套头文件

#include <sys/types.h>              // 2. C 系统头
#include <unistd.h>

#include <string>                   // 3. C++ 标准库
#include <vector>

#include "base/basictypes.h"        // 4. 其他库
#include "third_party/absl/flags/flag.h"

#include "foo/internal/helper.h"    // 5. 本项目
```

- 路径相对项目根目录，不出现 `.` / `..`。
- **用到什么就 include 什么**，不吃间接包含的红利 —— 哪怕 `foo.h` 已经包了 `bar.h`，`foo.cc` 用到 `bar` 的符号就得自己包一遍。
- 配套头文件排第一，是为了让 `foo2.h` 缺依赖时在 `foo.cc` 这边先炸，而不是炸在别人那里。
- **前向声明现在是不推荐的**（规范改过口径）。它隐藏依赖、挡住库作者改 API、给 `std::` 做前向声明直接 UB，还可能悄悄改变重载决议。默认 `#include`。
- `inline` 只给 10 行以内的小函数。带循环或 switch 的内联通常不划算；虚函数、递归函数本来也内联不了。析构函数比看起来长（成员和基类的析构是隐式的），别轻易内联。
- 头文件必须自给自足：自带 guard、自带依赖。做不到的（要插到别的文件中间的那种）改用 `.inc`，且尽量别用。不要用 `-inl.h`，也不要用 `#pragma once`。

## 作用域与生命周期

- 代码放进命名空间，顶层带项目名。**禁 `using namespace foo`**（using 指令），禁 inline namespace。`.cc` 里可以 `using ::foo::Bar`（using 声明）。命名空间内容不缩进，收尾写 `}  // namespace foo`。
- `.cc` 里对外不可见的东西放匿名命名空间或声明 `static`；`.h` 里不许这么干。
- 局部变量：作用域压到最小，**声明即初始化**。别在循环体里声明会反复构造析构的对象。
- **静态存储期的变量必须能平凡析构**。这条最容易踩：

```cpp
const int kNum = 10;                              // OK
constexpr std::array<int, 3> kArray = {1, 2, 3};  // OK
const char* const kMsg = "hello";                 // OK

const std::string kFoo = "foo";                   // 禁止：析构不平凡
static std::map<int, int> kData = {{1, 0}};       // 禁止：同上
static std::unique_ptr<Foo> g_foo;                // 禁止：析构要释放资源
```

  替代方案：查找表用平凡类型数组 + 线性/二分查找；真需要动态容器就用函数内局部静态指针并永不 delete（`static const auto& impl = *new T(args...);`）。常量初始化用 `constexpr` / `constinit` 标出来。

- `thread_local`：函数外定义的必须编译期常量初始化，用 `ABSL_CONST_INIT` 强制。函数内的没有初始化顾虑，但线程退出时的析构顺序会咬人 —— 析构函数里别碰其他 `thread_local`。

## 类与函数

| 事项 | 规则 |
|---|---|
| 构造函数 | 不调虚函数；不做可能失败又无法报错的初始化 → 改 `Init()` 或工厂函数 |
| 隐式转换 | 单参数构造函数、类型转换运算符一律 `explicit`；拷贝/移动构造和 `initializer_list` 构造除外 |
| 拷贝与移动 | 在 `public` 里显式 `= default` 或 `= delete`，明确表态"可拷贝 / 仅可移动 / 都不行"；拷贝构造与拷贝赋值必须成对，移动同理 |
| struct vs class | `struct` 只装被动数据，成员全 public、成员间无不变式约束；一旦有不变式或会长期演进就用 `class`；拿不准用 `class` |
| struct vs pair/tuple | 能起出有意义的名字就用 struct，别让读者去数 `.first` / `std::get<2>` |
| 继承 | 组合优先；只用 `public` 继承；只在 is-a 时继承；重写必须写 `override`（或 `final`），**重写时不要再写 `virtual`**；数据成员一律 private；避免多重实现继承 |
| 运算符重载 | 慎用，语义必须和内置运算符一致；不重载 `&&` `\|\|` `,` 和一元 `&`；**禁止自定义字面量 `operator""`**；不修改数据的二元运算符建议写成非成员函数；`operator==` 写成非成员 |
| 访问控制 | 数据成员全部 private（常量除外） |
| 声明次序 | `public:` → `protected:` → `private:`；每段内：类型和别名 → (struct 的)非静态数据成员 → 静态常量 → 工厂函数 → 构造与赋值 → 析构 → 其他函数 → 其他数据成员 |
| 函数长度 | 不设硬限，但超过 40 行就该想想能不能拆 |
| 输入输出 | 优先按值返回，其次按引用返回，裸指针只在可能为空时返回 |
| 参数 | 非可选输入 = 值或 `const&`；非可选输出/输入输出 = 引用；可选输入 = `std::optional` 或 `const` 指针；可选输出 = 非 const 指针。**所有输入参数排在输出参数前面**，新加参数也照这个位置塞，不是往末尾追加 |
| 重载 | 调用点必须一眼能看出调的是哪个；否则把参数信息写进函数名（`AppendString()` / `AppendInt()`） |
| 缺省参数 | 虚函数禁用；每个调用点取值可能不同的禁用；其余情况可读性明显更好才用，犹豫就改重载 |
| 后置返回类型 | 只在 lambda 或确实更好读时用，常规函数保持返回类型前置 |
| 所有权 | 动态对象要有单一固定的所有主；用 `std::unique_ptr` 表达所有权转移；`std::shared_ptr` 要有明确理由（通常是不可变对象 + 拷贝开销确实大）；禁 `std::auto_ptr` |

细则与完整代码示例见 `references/classes-functions.md`。

## 语言特性红绿灯

| 🔴 禁用 | 🟡 限制使用 | 🟢 推荐 |
|---|---|---|
| 异常（`throw` / `try`） | 宏：别在 `.h` 里定义，用完即 `#undef` | `const` 能加就加 |
| RTTI（`typeid`、`dynamic_cast`），单测例外 | 模板元编程：只在少数基础组件里用 | `constexpr` 定义真常量 |
| C 风格转换 `(int)x` | 友元：和被访问的类放同一文件 | `static_cast` / `const_cast` / `reinterpret_cast` |
| 流（`cout`/`cin`），仅日志接口例外 | 右值引用：只用于移动构造和移动赋值 | 指针用 `nullptr`，空字符用 `'\0'` |
| 变长数组、`alloca()` | `auto`：只用在局部变量 | `sizeof(varname)` 而非 `sizeof(type)` |
| 自定义字面量 `operator""` | lambda：可能逃逸当前作用域就显式捕获 | 迭代器和模板对象用前置 `++i` |
| `using namespace` / inline namespace | Boost：只用白名单子集 | 整型默认 `int`，定长用 `<stdint.h>` |
| 在 `std` 里声明东西、给 `std` 符号做前向声明 | 列表初始化：注意 `initializer_list` 构造函数会抢优先级 | 大整数用 `int64_t` |
| 列表初始化 `auto`（`auto d = {1.23}` 是 `initializer_list`） | | 用断言保证非负，**不要**用无符号类型表达"非负" |

每条的理由和替代方案见 `references/cpp-features.md`。无符号整型那条尤其值得讲清楚 —— `for (unsigned i = v.size()-1; i >= 0; --i)` 是个死循环。

## 格式

80 列 / 2 空格缩进 / 只用空格不用 tab / 左大括号不另起一行。

```cpp
class MyClass : public OtherClass {
 public:                                    // 访问控制符缩进 1 空格
  MyClass();                                // 成员缩进 2 空格
  explicit MyClass(int var);

  void set_some_var(int var) { some_var_ = var; }
  int some_var() const { return some_var_; }

 private:
  int some_var_;
};

ReturnType LongClassName::ReallyLongFunctionName(
    Type par_name1,                         // 换行参数缩进 4 空格
    Type par_name2) {
  if (condition) {                          // if 后有空格，括号内没有
    DoSomething();
  } else if (other) {                       // else 跟在 } 后面同一行
    DoOther();
  }

  for (int i = 0; i < kSomeNumber; ++i) {}  // 空循环体写 {} 或 continue

  switch (var) {
    case 0: {
      break;
    }
    default: {                              // 枚举没覆盖全时必须有 default
      assert(false);
    }
  }
  return result;                            // return 不加多余括号
}

MyClass::MyClass(int var)
    : some_var_(var),                       // 初始化列表放不下就缩进 4 空格
      some_other_var_(var + 1) {            // 多行时每成员一行、逐行对齐
  DoSomething();
}
```

几条容易漏的：

- `if-else` 的大括号要么都写要么都不写，不能一边有一边没有。
- 单行 `if` 只在没有 `else` 时允许：`if (x == kFoo) return new Foo();`
- 指针 `*` / 引用 `&` 靠类型还是靠变量名都行，但**同一文件内必须统一**；多重声明里不许用（`int x, *y;` 不行）。
- 布尔表达式断行时 `&&` 放行尾。
- 预处理指令顶格写，哪怕它在缩进的代码块里。
- 命名空间内容不缩进。
- 垂直留白越少越好，函数体首尾不留空行。

格式这块交给 `.clang-format` 最省事，配置见 `assets/clang-format`。剩下的细则在 `references/formatting.md`。

## 注释

- 统一用 `//`。每个文件开头写版权声明。
- **函数声明处写"做什么"**（叙述式：`Returns an iterator...`，不是祈使式 `Return an iterator`），**定义处写"怎么做、为什么这么做"**。不要把 `.h` 的注释复制到 `.cc`。
- 类注释说明功能、用法、线程安全前提，必要时附一小段用法示例。
- 变量能接受哨兵值（`nullptr`、`-1`）时必须注明含义。
- **注释解释 why，不翻译 what**。`// Find the element in the vector.` 配一句 `std::find` 是废话；能改成自解释的代码（`if (!IsAlreadyProcessed(element))`）就别写注释。
- `TODO(姓名或邮箱或 bug ID): 说明`，`DEPRECATED(姓名): 迁移指引`。

模板和更多例子见 `references/comments.md`。

## 回复前自查

- [ ] 成员变量带尾下划线了吗？常量是 `kXxx` 吗？函数是大驼峰吗？
- [ ] 新写的 `#include` 排进正确分组了吗？有没有依赖间接包含？
- [ ] 有没有新增非平凡析构的全局/静态变量？
- [ ] 单参数构造函数加 `explicit` 了吗？重写虚函数写 `override` 且去掉 `virtual` 了吗？
- [ ] 类的拷贝/移动能力有没有显式表态？
- [ ] 输出参数是不是排在输入参数后面？
- [ ] 有没有误用红灯特性（异常、RTTI、C 风格转换、流、`using namespace`）？
- [ ] 改的是别人的文件吗？如果是，有没有强行套新风格而破坏了原有一致性？

## 参考文件索引

| 文件 | 什么时候读 |
|---|---|
| `references/naming.md` | 命名拿不准、需要完整规则或边界情况 |
| `references/headers-scoping.md` | 头文件组织、依赖管理、全局/静态/thread_local 变量 |
| `references/classes-functions.md` | 设计类接口、拷贝移动语义、继承、参数传递 |
| `references/cpp-features.md` | "这个特性能不能用"、每条禁令的理由和替代方案 |
| `references/formatting.md` | 格式细节、换行方式、水平/垂直留白 |
| `references/comments.md` | 写文档注释、注释模板 |
| `references/variants.md` | 团队改编版（如知乎流传的国内版）与官方的差异、冲突怎么裁 |
| `assets/clang-format` | 给项目落地格式化配置（排版） |
| `assets/clang-tidy` | 给项目落地静态检查配置（命名 + 语言特性），含存量代码库的推进步骤 |
