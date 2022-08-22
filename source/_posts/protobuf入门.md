---
title: Protobuf入门
comments: true
keywords: 博客文章密码
message: 输入密码，查看文章
tags: Protobuf
abbrlink: 638e39a5
date: 2022-08-19 13:46:17
password:
abstract:
---

# 什么是Protobuf

在网络通信和通用数据交换等应用场景中经常使用的技术是 JSON 或 XML，还有一种类型Protobuf。

Protobuf是Protocol Buffers的简称，它是Google出品的性能优异、跨语言、跨平台的序列化库，用于描述一种轻便高效的结构化数据存储格式，并于2008年对外开源。Protobuf可以用于结构化数据串行化，或者说序列化。它的设计非常适用于在网络通讯中的数据载体，很适合做数据存储或 RPC 数据交换格式，它序列化出来的数据量少再加上以 K-V 的方式来存储数据，对消息的版本兼容性非常强，可用于通讯协议、数据存储等领域的语言无关、平台无关、可扩展的序列化结构数据格式。开发者可以通过 ProtoBuf 定义数据结构，然后通过 ProtoBuf 工具生成各种语言版本的数据结构类库，用于操作 ProtoBuf 协议数据

Protobuf是一种语言无关、平台无关、可扩展的序列化结构数据的方法，它可用于（数据）通信协议、数据存储等。

Protocol Buffers 是一种灵活，高效，自动化机制的结构数据序列化方法－可类比 XML，但是比 XML 更小（3 ~ 10倍）、更快（20 ~ 100倍）、更为简单。json、xml都是基于文本格式，protobuf是二进制格式。

Protobuf中最基本的数据单元是message，是类似Go语言中结构体的存在。在message中可以嵌套message或其它的基础数据类型的成员。

> 序列化(serialization、marshalling)的过程是指将数据结构或者对象的状态转换成可以存储(比如文件、内存)或者传输的格式(比如网络)。反向操作就是反序列化(deserialization、unmarshalling)的过程。
<!--more-->
![image-20220818145636178](https://raw.githubusercontent.com/gods-view/images/master/imageimage-20220818145636178.png)

# 定义Message

使用 ProtoBuf ，首先需要通过 ProtoBuf 语法定义数据结构(消息)，这些定义好的数据结构保存在.proto为后缀的文件中。

## 字段类型与字段编号

```protobuf
// 指定protobuf的版本，proto3是最新的语法版本，如果省略protocol buffer编译器默认使用proto2语法。他必须是文件中非空非注释行的第一行
syntax = "proto3";
// package 定义proto的包名，包名可以避免对message 类型之间的名字冲突，同名的Message可以通过package进行区分。

// 在没有为特定语言定义option xxx_package的时候，它还可以用来生成特定语言的包名，比如Java package, go package。

message Request {
  string userId = 1;   // 定义一个string类型的字段，字段名字为userId, 序号为1
  int32 operation = 2;   // 定义一个int32类型的字段，字段名字为operation, 序号为2
}

// 定义数据结构，message 你可以想象成Go语言中的struct
message Response {
  string data = 1;   // 定义一个string类型的字段，字段名字为data, 序号为1
  int32 status = 2;   // 定义一个int32类型的字段，字段名字为status, 序号为2
}

// 还可以给字段指定复合类型，包括枚举类型和其他message类型
```

字段是以`[ "repeated" ] type fieldName "=" fieldNumber [ "[" fieldOptions "]" ] ";"`格式定义的。这个例子是一个简单的例子，采用了`type fieldName "=" fieldNumber`格式定义的。

比如第一个字段userId, 首先是它的类型`string`，其次是字段的名称，然后是等号`=`, 之后是字段的序号，然后是分号。

复杂的结构，前面可以定义为`repeated`, 序号之后可以定义一些可选项。

这是普通的字段定义，当然还有一些复杂的一些字段定义，比如`Oneof`、`Map`、`Reserved`、`enum`定义，下一节我们再详细讲。

> 在message定义中每个字段都有一个唯一的编号，这些编号被用来在二进制消息体中识别你定义的这些字段，一旦你的message类型被用到后就不应该在修改这些编号了。注意在将message编码成二进制消息体时字段编号1-15将会占用1个字节，16-2047将占用两个字节。所以在一些频繁使用用的message中，你应该总是先使用前面1-15字段编号。
>
> 你可以指定的最小编号是1，最大是2E29 - 1（536,870,911）。其中19000到19999是给protocol buffers实现保留的字段标号，定义message时不能使用。

### 保留字段

当你删掉或者注释掉message中的一个字段时，未来其他开发者在更新message定义时就可以重用之前的字段编号。如果他们意外载入了老版本的`.proto`文件将会导致严重的问题，比如数据损坏、隐私泄露等。一种避免问题发生的方式是指定保留的字段编号和字段名称。如果未来有人用了这些字段标识那么在编译时protocol buffer的编译器会报错。

```protobuf
message Foo {
  reserved 2, 15, 9 to 11;  // 保留字段
  reserved "foo", "bar";
}
```

### Optional的字段和默认值

如上所述，消息描述中的一个元素可以被标记为“可选的”（optional）。一个格式良好的消息可以包含0个或一个optional的元素。当解 析消息时，如果它不包含optional的元素值，那么解析出来的对象中的对应字段就被置为默认值。默认值可以在消息描述文件中指定。例如，要为 *SearchRequest*消息的*result_per_page*字段指定默认值10，在定义消息格式时如下所示：

```protobuf
optional int32 result_per_page = 3 [default = 10];
```

如果没有为optional的元素指定默认值，就会使用与特定类型相关的默认值：对string来说，默认值是空字符串。对bool来说，默认值是false。对数值类型来说，默认值是0。对枚举来说，默认值是枚举类型定义中的第一个值。

### 枚举

当需要定义一个消息类型的时候，可能想为一个字段指定某“预定义值序列”中的一个值。例如，假设要为每一个SearchRequest消息添加一个 corpus字段，而corpus的值可能是UNIVERSAL，WEB，IMAGES，LOCAL，NEWS，PRODUCTS或VIDEO中的一个。 其实可以很容易地实现这一点：通过向消息定义中添加一个枚举（enum）就可以了。一个enum类型的字段只能用指定的常量集中的一个值作为其值（如果尝 试指定不同的值，解析器就会把它当作一个未知的字段来对待）。在下面的例子中，在消息格式中添加了一个叫做Corpus的枚举类型——它含有所有可能的值 ——以及一个类型为Corpus的字段：

```protobuf
message SearchRequest {
  required string query = 1;
  optional int32 page_number = 2;
  optional int32 result_per_page = 3 [default = 10];
  enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
  }
  optional Corpus corpus = 4 [default = UNIVERSAL];
}
```

你可以为枚举常量定义别名。 需要设置allow_alias option 为 true, 否则 protocol编译器会产生错误信息。

```protobuf
enum EnumAllowingAlias {
  option allow_alias = true;
  UNKNOWN = 0; // 枚举类型的第一个选项的标识符必须是0，这也是枚举类型的默认值。
  STARTED = 1;
  RUNNING = 1; // 如果设置allow_alias，允许字段编号重复，RUNNING是STARTED的别名。
}
enum EnumNotAllowingAlias {
  UNKNOWN2 = 0;
  STARTED2 = 1;
  // RUNNING = 1;  // Uncommenting this line will cause a compile error inside Google and a warning message outside.
}
```

枚举常量必须在32位整型值的范围内。因为enum值是使用可变编码方式的，对负数不够高效，因此不推荐在enum中使用负数。如上例所示，可以在 一个消息定义的内部或外部定义枚举——这些枚举可以在.proto文件中的任何消息定义里重用。当然也可以在一个消息中声明一个枚举类型，而在另一个不同 的消息中使用它——采用MessageType.EnumType的语法格式。

当对一个使用了枚举的.proto文件运行protocol buffer编译器的时候，生成的代码中将有一个对应的enum（对Java或C++来说），或者一个特殊的EnumDescriptor类（对 Python来说），它被用来在运行时生成的类中创建一系列的整型值符号常量（symbolic constants）。

注意枚举类型的定义采用C++ scoping规则，也就是枚举值是枚举类型的兄弟类型，而不是子类型，所以避免在同一个package定义重名的枚举字段。

### Oneof

如果你有一组字段，同时最多允许这一组中的一个字段出现，就可以使用`Oneof`定义这一组字段，这有点Union的意思，但是Oneof允许你设置零各值。

因为proto3没有办法区分正常的值是否是设置了还是取得缺省值(比如int64类型字段，如果它的值是0，你无法判断数据是否包含这个字段，因为0几可能是数据中设置的值，也可能是这个字段的零值)，所以你可以通过Oneof取得这个功能，因为Oneof有判断字段是否设置的功能。

```protobuf
syntax = "proto3";

package abc;

message OneofMessage {
    oneof test_oneof {
      string name = 4;
      int64 value = 9;
    }
  }
```

`oneof`字段不能同时使用`repeated`。

### map类型

map类型需要设置键和值的类型，格式是`"map" "<" keyType "," type ">" mapName "=" fieldNumber [ "[" fieldOptions "]"`。

比如:

```protobuf
map<int64,string> values = 1;
```

`map`字段不能同时使用`repeated`。

### Any

`Any`字段允许你处理嵌套数据，并不需要它的proto定义。一个`Any`以bytes呈现序列化的消息，并且包含一个URL作为这个类型的唯一标识和元数据。

为了使用`Any`类型，你需要引入`google/protobuf/any.proto`。

```protobuf
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated google.protobuf.Any details = 2;
}
```

Any类型用来替换proto2中的扩展。

### 使用其他消息类型

你可以将其他消息类型用作字段类型。例如，假设在每一个SearchResponse消息中包含Result消息，此时可以在相同的.proto文件中定义一个Result消息类型，然后在SearchResponse消息中指定一个Result类型的字段，如：

```protobuf
message SearchResponse {
  repeated Result result = 1;
}

message Result {
  required string url = 1;
  optional string title = 2;
  repeated string snippets = 3;
}
```

### 导入定义

在上面的例子中，Result消息类型与SearchResponse是定义在同一文件中的。如果想要使用的消息类型已经在其他.proto文件中已经定义过了呢？
你可以通过导入（importing）其他.proto文件中的定义来使用它们。要导入其他.proto文件的定义，你需要在你的文件中添加一个导入声明，如：

```protobuf
import "myproject/other_protos.proto";
```

默认情况下你只能使用直接导入的.proto文件中的定义. 然而， 有时候你需要移动一个.proto文件到一个新的位置， 可以不直接移动.proto文件， 只需放入一个dummy .proto 文件在老的位置， 然后使用import转向新的位置:

```protobuf
// new.proto
// All definitions are moved here
// old.proto
// This is the proto that all clients are importing.
import public "new.proto";
import "other.proto";
```

// client.proto

```protobuf
import "old.proto";
// You use definitions from old.proto and new.proto, but not other.proto
```

protocol编译器就会在一系列目录中查找需要被导入的文件，这些目录通过protocol编译器的命令行参数-I/–import_path指定。如果不提供参数，编译器就在其调用目录下查找。

### 嵌套类型

你可以在其他消息类型中定义、使用消息类型，在下面的例子中，Result消息就定义在SearchResponse消息内，如：

```protobuf
message SearchResponse {
  message Result {
    required string url = 1;
    optional string title = 2;
    repeated string snippets = 3;
  }
  repeated Result result = 1;
}
```

如果你想在它的父消息类型的外部重用这个消息类型，你需要以Parent.Type的形式使用它，如：

```protobuf
message SomeOtherMessage {
  optional SearchResponse.Result result = 1;
}
```

当然，你也可以将消息嵌套任意多层，如：

```protobuf
message Outer {                  // Level 0
  message MiddleAA {  // Level 1
    message Inner {   // Level 2
      required int64 ival = 1;
      optional bool  booly = 2;
    }
  }
  message MiddleBB {  // Level 1
    message Inner {   // Level 2
      required int32 ival = 1;
      optional bool  booly = 2;
    }
  }
}
```

### 更新一个消息类型

==如果一个已有的消息格式已无法满足新的需求——如，要在消息中添加一个额外的字段——但是同时旧版本写的代码仍然可用。不用担心！更新消息而不破坏已有代码是非常简单的。在更新时只要记住以下的规则即可。==

- 不要更改任何已有的字段的数值标识。
  所添加的任何字段都必须是optional或repeated的。这就意味着任何使用“旧”的消息格式的代码序列化的消息可以被新的代码所解析，因为它们 不会丢掉任何required的元素。应该为这些元素设置合理的默认值，这样新的代码就能够正确地与老代码生成的消息交互了。类似地，新的代码创建的消息 也能被老的代码解析：老的二进制程序在解析的时候只是简单地将新字段忽略。然而，未知的字段是没有被抛弃的。此后，如果消息被序列化，未知的字段会随之一 起被序列化——所以，如果消息传到了新代码那里，则新的字段仍然可用。注意：对Python来说，对未知字段的保留策略是无效的。
- 非required的字段可以移除——只要它们的标识号在新的消息类型中不再使用（更好的做法可能是重命名那个字段，例如在字段前添加“OBSOLETE_”前缀，那样的话，使用的.proto文件的用户将来就不会无意中重新使用了那些不该使用的标识号）。
- 一个非required的字段可以转换为一个扩展，反之亦然——只要它的类型和标识号保持不变。
- int32, uint32, int64, uint64,和bool是全部兼容的，这意味着可以将这些类型中的一个转换为另外一个，而不会破坏向前、 向后的兼容性。如果解析出来的数字与对应的类型不相符，那么结果就像在C++中对它进行了强制类型转换一样（例如，如果把一个64位数字当作int32来 读取，那么它就会被截断为32位的数字）。
- sint32和sint64是互相兼容的，但是它们与其他整数类型不兼容。
- string和bytes是兼容的——只要bytes是有效的UTF-8编码。
- 嵌套消息与bytes是兼容的——只要bytes包含该消息的一个编码过的版本。
- fixed32与sfixed32是兼容的，fixed64与sfixed64是兼容的。

## proto文件编译

将.proto文件，编译成指定语言类库

### 安装Protobuf编译器

protobuf的github发布地址： https://github.com/protocolbuffers/protobuf/releases

protobuf的编译器叫protoc，在上面的网址中找到最新版本的安装包，下载安装。

1. `unzip protoc-****-osx-x86_64.zip`
2. `cp -r include/ /usr/local/include/  # 一个"/"都不能少`
3. `cp -r bin/ /usr/local/bin/  # 一个"/"都不能少`

打开cmd，命令窗口执行protoc命令，没有报错的话，就已经安装成功。

### 安装protoc-gen-go插件

Protobuf核心的工具集是C++语言开发的，官方的protoc编译器中并不支持Go语言，需要安装一个插件才能生成Go代码。用如下命令安装：

`go get -u -v github.com/golang/protobuf/protoc-gen-go@v1.3.0 // 指定protoc-gen-go的版本`

提供了一个`protoc-gen-go`二进制文件，当编译器调用时传递了`--go_out`命令行标志时`protoc`就会使用它。`--go_out`告诉编译器把Go源代码写到哪里。编译器会为每个`.proto`文件生成一个单独的源代码文件。

### 编译对应语言的pb文件

在当前的目录下执行`protoc -I=. -I/usr/local/include -I=$(GOPATH)/src --go_out=. simple.proto`, 可以将这个proto编译成Go的代码，因为这里我们使用了`go_out`输出格式。

`-I`指定protoc的搜索import的proto的文件夹。在`MacOS`操作系统中protobuf把一些扩展的proto放在了`/usr/local/include`对应的文件夹中，一些第三方的Go库放在了gopath对应的包下，所以这里都把它们加上了。对于这个简单的例子，实际是不需要的。

`cpp_out`用来生成C++代码，`java_out`产生Java代码，`python_out`产生python代码，类似地还有`csharp_out`、`objc_out`、`ruby_out`、`php_out`等参数。

一些第三方的插件也会定义自己的输出插件，比如`gofast_out`使用gogo库生成代码， `rust_out`产生rust代码。

生成的代码我们指定放在本地文件夹中(`--go_out=.`)。

输出文件的名称是通过获取.proto文件的名称并进行两处更改来计算的：

- 生成文件的扩展名是`.pb.go`。比如说`user.proto`编译后会得到`user.pb.go`。
- proto路径（使用`--proto_path`或`-I`命令行标志指定）将替换为输出路径（使用`--go_out`标志指定）。

当你运行如下编译命令时：

```shell
protoc --proto_path=src --go_out=build/gen src/foo.proto src/bar/baz.proto
```

编译器会读取文件`src/foo.proto`和`src/bar/baz.proto`，这将会生成两个输出文件`build/gen/foo.pb.go`和`build/gen/bar/baz.pb.go`

如果有必要，编译器会自动生成`build/gen/bar`目录，但是他不能创建`build`或者`build/gen`目录，这两个必须是已经存在的目录。

### 包

如果一个`.proto`文件中有包声明，生成的源代码将会使用它来作为Go的包名，如果`.proto`的包名中有`.` 在Go包名中会将`.`转换为`_`。举例来说`proto`包名`example.high_score`将会生成Go包名`example_high_score`。

在`.proto`文件中可以使用option `go_package`指令来覆盖上面默认生成Go包名的规则。比如说包含如下指令的一个`.proto`文件

```protobuf
package example.high_score;
option go_package = "hs";
```

生成的Go源代码的包名是`hs`。

如果一个`.proto`文件中不包含package声明，生成的源代码将会使用`.proto`文件的文件名(去掉扩展名)作为Go包名，`.`会被首先转换为`_`。举例来说一个名为`high.score.proto`不包含pack声明的文件将会生成文件`high.score.pb.go`，他的Go包名是`high_score`。

### 消息

一个简单的消息声明：

```protobuf
message Foo {}
```

protocol buffer编译器将会生成一个名为`Foo`的结构体，实现了`proto.Message`接口的`Foo`类型的指针

```protobuf
type Foo struct {
}

// 重置proto为默认值
func (m *Foo) Reset()         { *m = Foo{} }

// String 返回proto的字符串表示
func (m *Foo) String() string { return proto.CompactTextString(m) }

// ProtoMessage作为一个tag 确保其他人不会意外的实现
// proto.Message 接口.
func (*Foo) ProtoMessage()    {}
```

### 内嵌的消息

一个message可以声明在其他message的内部。比如说：

```protobuf
message Foo {
  message Bar {
  }
}
```

这种情况，编译器会生成两个结构体：`Foo`和`Foo_Bar`。

### 字段

编译器会为每个在message中定义的字段生成一个Go结构体的字段，字段的确切性质取决于它的类型以及它是`singular`，`repeated`，`map`还是`oneof`字段。

注意生成的Go结构体的字段将始终使用驼峰命名，即使在`.proto`文件中消息字段用的是小写加下划线（应该这样）。大小写转换的原理如下：

- 首字母会大些，如果message中字段的第一个字符是`_`，它将被替换为X。
- 如果内部下划线后跟小写字母，则删除下划线，并将后面跟随的字母大写。

因此，proto字段`foo_bar_baz`在Go中变成`FooBarBaz`， `_my_field_name_2`变为`XMyFieldName_2`。

### 单一标量字段

对于字段定义：

```protobuf
int32 foo = 1;
```

编译器将生成一个带有名为Foo的int32字段和一个访问器方法GetFoo（）的结构，该方法返回Foo中的int32值或该字段的零值（如果字段未设置（数值型零值为0，字符串为空字符串））。

### 单一message字段

给出如下消息类型

```protobuf
message Bar {}
```

对于一个有`Bar`类型字段的消息：

```protobuf
// proto3
message Baz {
  Bar foo = 1;
}
```

编译器将会生成一个Go结构体

```Go
type Baz struct {
        Foo *Bar
}
```

消息类型的字段可以设置为nil，这意味着该字段未设置，有效清除该字段。这不等同于将值设置为消息结构体的“空”实例。

编译器还生成一个`func（m * Baz）GetFoo（）* Bar`辅助函数。这让不在中间检查nil值进行链式调用成为可能。

### 可重复字段

每个重复的字段在Go中的结构中生成一个T类型的slice，其中T是字段的元素类型。对于带有重复字段的此消息：

```protobuf
message Baz {
  repeated Bar foo = 1;
}
```

编译器会生成如下结构体：

```go
type Baz struct {
        Foo  []*Bar
}
```

同样，对于字段定义`repeated bytes foo = 1;`编译器将会生成一个带有类型为`[][]byte`名为`Foo`的字段的Go结构体。对于可重复的枚举`repeated MyEnum bar = 2;`，编译器会生成带有类型为`[]MyEnum`名为`Bar`的字段的Go结构体。

### 映射字段

每个映射字段会在Go的结构体中生成一个`map[TKey]TValue`类型的字段，其中`TKey`是字段的键类型`TValue`是字段的值类型。对于下面这个消息定义：

```protobuf
message Bar {}

message Baz {
  map<string, Bar> foo = 1;
}
```

编译器生成Go结构体

```go
type Baz struct {
        Foo map[string]*Bar
}
```

## 枚举

给出如下枚举

```protobuf
message SearchRequest {
  enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
  }
  Corpus corpus = 1;
  ...
}
```

编译器将会生成一个枚举类型和一系列该类型的常量。

对于消息中的枚举（像上面那样），类型名字以消息名开头

```go
type SearchRequest_Corpus int32
```

对于包级别的枚举：

```protobuf
// .proto
enum Foo {
  DEFAULT_BAR = 0;
  BAR_BELLS = 1;
  BAR_B_CUE = 2;
}
```

Go 中的类型不会对proto中的枚举名称进行修改：

```go
type Foo int32
```

此类型具有`String()`方法，该方法返回给定值的名称。

`Enum()`方法使用给定值初始化新分配的内存并返回相应的指针：

```go
func (Foo) Enum() *Foo
```

编译器为枚举中的每个值生成一个常量。对于消息中的枚举，常量以消息的名称开头：

```go
const (
        SearchRequest_UNIVERSAL SearchRequest_Corpus = 0
        SearchRequest_WEB       SearchRequest_Corpus = 1
        SearchRequest_IMAGES    SearchRequest_Corpus = 2
        SearchRequest_LOCAL     SearchRequest_Corpus = 3
        SearchRequest_NEWS      SearchRequest_Corpus = 4
        SearchRequest_PRODUCTS  SearchRequest_Corpus = 5
        SearchRequest_VIDEO     SearchRequest_Corpus = 6
)
```

对于包级别的枚举，常量以枚举名称开头:

```go
const (
        Foo_DEFAULT_BAR Foo = 0
        Foo_BAR_BELLS   Foo = 1
        Foo_BAR_B_CUE   Foo = 2
)
```

protobuf编译器还生成从整数值到字符串名称的映射以及从名称到值的映射：

```go
var Foo_name = map[int32]string{
        0: "DEFAULT_BAR",
        1: "BAR_BELLS",
        2: "BAR_B_CUE",
}
var Foo_value = map[string]int32{
        "DEFAULT_BAR": 0,
        "BAR_BELLS":   1,
        "BAR_B_CUE":   2,
}
```

请注意，`.proto`语言允许多个枚举符号具有相同的数值。具有相同数值的符号是同义词。这些在Go中以完全相同的方式表示，多个名称对应于相同的数值。反向映射包含数字值的单个条目，数值映射到出现在`proto`文件中首先出现的名称。