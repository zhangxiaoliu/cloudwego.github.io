---
title: 'hz access plugin'
date: 2023-01-21
weight: 3
description: >
---

Currently, hz code generation is based on the plugin model of "thriftgo" and "protoc", which is a great help in accessing some third party plugins, especially for "protoc" which currently supports a rich plugin ecosystem.

So hz provides the means to extend third-party plug-ins.

## ThriftGo Plugin

### Usage

```shell
hz new  --idl={YOUR-IDL.thrift} --thrift-plugins={PLUGIN-NAME}
```

If the plugin needs to pass some options, they are as follows:

```shell
hertztool new --idl={YOUR-IDL.thrift} --thrift-plugins={PLUGIN-NAME}:{YOUR-OPTION1,YOUR-OPTION2}
```

### Example

Currently, `thriftgo` provides a plugin "thrift-gen-validator" for generating structural parameter validation functions that can be generated along with the model.

- Install : `go install github.com/cloudwego/thrift-gen-validator@latest`

- Use : `hz new --idl=idl/hello.thrift --thrift-plugins=validator`

- Code : [code](https://github.com/cloudwego/hertz-examples/tree/main/hz/plugin/thrift)

## Protoc Plugin

### go_package Detail

The path to the go file generated by the Protoc plugin depends on the definition of the "go_package" option. So let's take a closer look at the role of "go_package":
"option go_package" in the proto file refers to the import location of the go file generated by this file, as follows

If the go_package option is defined this way:

```protobuf
option go_package = "github.com/a/b/c";
```

If you then want to cite the content that generated it, you can do so as follows:

```go
import "github.com/a/b/c"

var req c.XXXStruct
```

In addition, the "option go_package" is also related to the path to the generated code. It is as follows:

If the option go_package is defined this way:

```protobuf
// psm.proto
option go_package = "github.com/a/b/c";
```

The generated go file will be located at: github.com/a/b/c/psm.pb.go

In addition, "option go_package" can be used in conjunction with the go module as follows

If the option go_package is defined like this:

```protobuf
// psm.proto
option go_package = "github.com/a/b/c";
```

Initialize the go module

```shell
go mod init github.com/a/b
```

Generate code and specify module

```shell
protoc go_out=. --go_opt=module=github.com/a/b psm.proto
```

Then the generated file will be: c/psm.pb.go

However, since the module for this project is "github.com/a/b"; so, if you refer to the generated content of this file,

the import path would still be "github.com/a/b/c"

### Usage

Currently, hz is doing some processing of "go_package" in order to unify the generated models, with the following rules:

Suppose the current project is github.com/a/b:

- go_package="github.com/a/b/c/d": the code will be generated under "/biz/model/c/d"; (no change)
- go_package="github.com/a/b/biz/model/c/d": will generate the model under "/biz/model/c/d", where "biz/model" is the default model generation path, which can be changed with the "-model_dir" option.
- go_package="x/y/z": will generate the code under "biz/model/x/y/z" (relative path completion) (no change).
- go_package="google.com/whatever": no code (external IDL) will be generated (no change).
- go_package="github.com/a/b/c": code will be generated under "biz/model/c" (no change).

So you can simply define a "go_package" such as "{$MODULE}/{$MODEL_DIR}/x/y/z" (where {$MODEL_DIR} defaults to "biz/model", or you can use the "model_dir" option to define it) to enable access to third-party plugins on the hz side.

Use the command :

```shell
hertztool new --idl={YOUR-IDL.proto} --protoc-plugins={PLUGIN_NAME}:{OPTION1,OPTION2}:{OUT_DIR} --mod={YOUR_MODULE}
```

### Example

Here is an example of using the Hz integration `protoc-gen-openapi` plugin to generate openapi 3.0 documentation.

- Install :`go install github.com/google/gnostic/cmd/protoc-gen-openapi@latest`

- Define go_package for idl: "middleware/hertz/biz/model/psm"

- Usage : `hz new -I=idl --idl=idl/hello/hello.proto --protoc-plugins=openapi::./docs --mod=middleware/hertz`

- Code : [code](https://github.com/cloudwego/hertz-examples/tree/main/hz/plugin/proto)