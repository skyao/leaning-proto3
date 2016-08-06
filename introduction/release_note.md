Proto3.0.0 Release Note
===============

> 注： 以下内容翻译自 [Protocol Buffers v3.0.0 的官方 release note](https://github.com/google/protobuf/releases)


这个修改日志总结了从最后一个稳定版本(v2.6.1)以来所有的修改。从 v3.0.0-beta-4 之后的修改请见最后一节。

## Proto3

- 引入 Protocol Buffers 语言版本 3 (称为 proto3)

	当 protocol buffers 最初开源时，它实现了 Protocol Buffers 语言版本 2 (称为 proto2), 这也是为什么版本数从 v2.0.0 开始。从 v3.0.0 开始， 引入新的语言版本(proto3)，而旧的版本(proto2)继续被支持。

    引入proto3的主要意图是在将这个语言推向 google 新的API平台之前清理 protobuf。在 proto3 中，语言被简化，即便于使用又可以用于更大范围的编程语言。同时添加了一些新的特性来更好的支持 API 中的通用习语。

    下面是语言版本3中主要的新特性：

    1. 移除用于原生值字段的字段表述逻辑，移除必填(required)字段，并移除默认值。这显著的简化了 proto3  在实现开放结构标示(open struct representations)中实现，例如在如Android Java, Objective C, 或者 Go 语言中。
    2. 移除未知字段
    3. 移除扩展(extensions)，替代为新的称为 Any 的标准类型
    4. 修复未知枚举值的语义
    5. 此外还有 map (向后移植到proto2)
    6. 此外还有少量用于表述时间，动态日志等的标准类型 (向后移植到proto2)
    7. 定义良好的JSON编码，作为二进制编码之外的备选

	引入了一个新的概念 "syntax" 来指定一个.proto文件使用的是 proto2 还是 proto3：

    ```java
    // foo.proto
    syntax = "proto3";
    message Bar {...}
    ```

	如果没有设置，protocol buffer编译器将生成警告而"proto2"会被默认使用。在未来版本中这个警告将可能变成错误。

    我们推荐新的Protocol Buffers 用户使用proto3.当然，我们不普遍推荐现有用户从proto2迁移到proto3,因为API不兼容，而我们将继续长期支持proto2.

    proto3中的其他重要改动：

- 在proto3语法中显式的 "optional" 关键字被禁止，因为字段默认就是可选的;必填字段不再被支持
- 删除非零默认值，而用于非message字段的字段表述逻辑如 has_xxx() 方法被删除;设置为默认值(对于数字字段是0,字符串/字节字段是空)的原生字段在序列化时被跳过
- 在proto3语法中group字段不再支持
- 在proto3中默认修改重复原生字段来支持打包序列化（当前版本中在C++, Java, Python中实现）。用户依然可以通过设置packed为false来禁用打包序列化
- 添加众所周知的类型 protos (any.proto, empty.proto, timestamp.proto, duration.proto 等).用户可以导入并使用这些protos，就像正规的proto文件一样。额外的运行时支持在每个语言中都可用。
- Proto3 JSON在一些语言中 (在C++, Java, Python 和 C# 中被完全支持， 在 Ruby 中被部分支持)被支持。 JSON 规范在proto3语言指南中被定义：

	https://developers.google.com/protocol-buffers/docs/proto3#json

	我们将发布更详细的规范来定义 proto3-顺应 的JSON序列器和解析器的确切行为。在此之前，请不要依赖实现的特定行为，如果他们没有在上面的规范中注明。

- Proto3强制严格 UTF-8 检查。如果字符串字段包含非UTF-8数据则解析将失败。

## 普遍

- 因为新的语言实现 (C#, JavaScript, Ruby, Objective-C)到proto3
- 添加对 map 字段的支持(同时在proto2和proto3中实现)。

	map字段可以使用下面的语法声明：

    ```java
    message Foo {
      map<string, string> values = 1;
    }
    ```

	map字段的数据在内存中被存储为未排序的map并可以通过生成的访问器来访问。

- 在 proto2 和 proto3 语法中同时添加 "reserved" (保留) 关键字。用户可以用这个关键字来声明保留字段数字和名称来防止他们被在同一个消息中的其他字段重用。

	为了保留字段数字，在消息中添加保留声明：

    ```java
    message TestMessage {
		reserved 2, 15, 9 to 11, 3;
    }
    ```

	这将保留字段数字  2, 3, 9, 10, 11 and 15。如果用户使用他们中的任何一个作为字段数字，protocol buffer 编译器将报告错误。

	字段名字也可以被保留：

    ```java
    message TestMessage {
		reserved "foo", "bar";
    }
    ```

- 添加一个确定性的序列化API(当前在c++中可用)。这个确定性序列化保证：给定的一个二进制，等同的消息将被序列化为同样的字节。这容许应用如MapReduce基于被序列化后的字节来分组相等的消息。这个确定性序列化 **不** 跨语言。在使用schema跨越不同build时也是不稳定的。需要权威序列化的用户，如以权威形式永久保存，指纹等，应该定义自己的权威性规范并使用反射API来实现序列器，而不是依赖这个API。
- 添加新的字段选项"json_name"。默认在proto3的JSON格式中 proto 字段名将被转换为"lowerCamelCase"(小写开头的驮峰法)。这个选项被用于覆盖这个行为并为这个字段指定不同的JSON名字。
- 添加一致性测试来确保实现遵循 proto3 JSON 规范

## c++

略过。

## Java

- 引入新的工具包，作为单独的artifact在maven中发布。它包括：

	* JsonFormat： 转换 proto message 到/从 JSON
	* Timestamps/Durations: 工具类功能来处理 Timestamp 和 Duration
	* FieldMaskUtil： 工具类功能来处理 FieldMask

- 引入 ExperimentalApi 注解。被注解的API是实现性的并且在未来的发布中以不兼容的方式修改。
- 引入 zero-copy 序列化作为 ExperimentalApi

	* 引入 `ByteOutput` 接口。这类似 `OutputStream` 但是为不可变的字段提供延迟写入的语义(例如不需要立即复制)。
	* `ByteString` 现在支持写入到 `ByteOutput`，这将直接暴露 `ByteString` 的内部 (如 ``byte[] 或者 `ByteBuffer`)到 `ByteOutput` 而无需复制
	* `CodedOutputStream` 现在支持写入到 `ByteOutput`。太大从而无法填充内部buffer的 `ByteString` 实例将被(延迟)直接写入到 `ByteOutput` 。
	* 这将容许应用使用大型 `ByteString` 字段来完全地避免这些字段的复制。这样应用可以提供 `ByteOutput` 来将从 `CodedOutputStream` 接收到的chunks链起来，在将他们发送给IO系统前。

- 其他 `CodedOutputStream` 的相关改变

	* `sun.misc.Unsafe` 的额外使用，使得可以实现对 `byte[]` 和 `ByteBuffer` 值的快速访问，并避免不必要的访问检查
	* 基于 `ByteBuffer` 的 `CodedOutputStream` 现在直接写入到 `ByteBuffer`，而不是中间数组

- 为字符串字段序列化而做的性能优化
- 每个被生成的消息中的静态 PARSER 被弃用， 并且将在未来版本中删除。替代的是为每个消息类型生成的静态 parser() getter方法。
- 文件选项  "java_generate_equals_and_hash" 现在被弃用。默认将生成  equals() 和 hashCode() 方法。

## Python / Ruby / Objective-C / C#

略过。

## JavaScript

- 为javascript添加 proto2/proto3 支持。运行时被使用纯javascript编写，并可以在浏览器和node.js中工作。为了给proto生成javascript代码，在调用 protoc 时带上 "--js_out"。查看 js/README.md 来获取更多构建建议。
- javascript已经支持二进制 protobuf 格式，但是不支持 proto3 JSON。也不支持反射，因为为此而需要的代码大小对浏览器而言通常不是一个正确的选择。
- 同时支持CommonJS导入和 Closure goog.require()。

## Lite

- 在为移动平台的java中支持 proto3 轻运行时

	在protoc中引入新的"lite"生成器参数来为c++ 为Proto3语法消息。使用案例：

    ```bash
    ./protoc --cpp_out=lite:$OUTPUT_PATH foo.proto
    ```

	protoc 将把当前输入和所有传递的依赖作为LITE。同样的生成器参数必须用于生成依赖。

    在proto3 语法文件中， "optimized_for=LITE_RUNTIME" 不再支持。

    对于Java，--javalite_out 代码生成器被支持在单独的分支中作为单独的编译器插件。

- 用于 android 上的 Java Lite 运行时的性能优化

	* 减少分配
	* 减少混淆后的方法overhead
	* 减少overhead后的代码大小

- Java Lite protos 现在实现了深度 equals/hashCode/toString

## 兼容性通知

- v3.0.0 是v3.× 系列的第一个稳定版本。未来不会有任何打破API的改动。
- 对于 C++, Java Lite 和 Objective-C，保证源码级别的兼容性。从v3.0.0升级到更新的小版本将会是源码兼容。例如，如果你的代码以 protobuf v3.0.0 编译，在升级 protobuf 类库到v3.1.0之后它将可以继续编译。
- 对于其他语言，源码级别兼容性和二进制级别兼容性同时被保证。例如，如果你有 protobuf v3.0.0的java二级值构建。在替换protobuf运行时类库到v3.1.0之后，之前构建的二进制将依然可以工作。
- 兼容性仅仅保证正式的API和正式的行为。如果使用非正式的API（例如 使用在c++ internal 命名空间中的任何东西），它可能以不确定的方式被小版本发布打破。

## Changes since v3.0.0-beta-4

略过。







