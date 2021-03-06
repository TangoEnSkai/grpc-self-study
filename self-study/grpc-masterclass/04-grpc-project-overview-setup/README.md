# Session 04: [Hands-On] gRPC Project Overview & Setup

---

## 13. Project Setup (Golang + VSCode + Protoc)

### 13.1. Go Setup

* Instructions to install Golang: https://golang.org/doc/install
* Make sure that the go binaries are in your `PATH`
  * (see installation instructions), so that you can run the following command in a terminal:

```bash
go version
go version go1.12.6 darwin/amd64
```

### 13.2. VSCode Setup

* instal VSCode: https://code.visualstudio.com/
* instal VSCode extensions [link](https://code.visualstudio.com/docs/editor/extension-gallery#_browse-and-install-extensions)
  * `vscode-proto3`
  * `Clang-Format`
    * MacOSX: `brew install clang-format`
* instal `protoc` (see below):

### 13.3. Protoc Setup

* in order to perform code generation, you will need to instal `protoc` on your computer
  * open a command line interface and type `brew install protobuf`

---

## 14. Go Dependencies Setup

* visit `gRPC-GO` github: [link](https://github.com/grpc/grpc-go)
* and follow the installation instructions:
  * it takes some time

```bash
go get -u google.golang.org/grpc
```

* visit `golang/protobuf` github: [link](https://github.com/golang/protobuf)
* run this command:

```bash
go get -u github.com/golang/protobuf/protoc-gen-go
```

---

## 15. Code Generation Test

* `go`directory` has
  * `bin`
  * `pkg`
  * `src`: the code written in Go is here

* create directories like this:
  * pwd/calculator/
  * pwd/greet/greetpb/greet.proto

* here is a dummy `.proto` file to test code generation works or not

```proto
syntax = "proto3";

package greet;
option go_package="greetpb";

service GreetService{}
```

* run this at `pwd`:

```bash
protoc greet/greetpb/greet.proto --go_out=plugins-grpc:.
```

* then you can see `greet.pb.go` has been creaetd under `greetpb`, where `greet.proto` is located.
* here is how `greet.pb.go` looks like:

```go
// Code generated by protoc-gen-go. DO NOT EDIT.
// source: greet/greetpb/greet.proto

package greetpb

import (
  context "context"
  fmt "fmt"
  proto "github.com/golang/protobuf/proto"
  grpc "google.golang.org/grpc"
  math "math"
)

// Reference imports to suppress errors if they are not otherwise used.
var _ = proto.Marshal
var _ = fmt.Errorf
var _ = math.Inf

// This is a compile-time assertion to ensure that this generated file
// is compatible with the proto package it is being compiled against.
// A compilation error at this line likely means your copy of the
// proto package needs to be updated.
const _ = proto.ProtoPackageIsVersion3 // please upgrade the proto package

func init() { proto.RegisterFile("greet/greetpb/greet.proto", fileDescriptor_fe6f881da19a2871) }

var fileDescriptor_fe6f881da19a2871 = []byte{
  // 72 bytes of a gzipped FileDescriptorProto
  0x1f, 0x8b, 0x08, 0x00, 0x00, 0x00, 0x00, 0x00, 0x02, 0xff, 0xe2, 0x92, 0x4c, 0x2f, 0x4a, 0x4d,
  0x2d, 0xd1, 0x07, 0x93, 0x05, 0x49, 0x10, 0x5a, 0xaf, 0xa0, 0x28, 0xbf, 0x24, 0x5f, 0x88, 0x15,
  0xcc, 0x31, 0xe2, 0xe3, 0xe2, 0x71, 0x07, 0x31, 0x82, 0x53, 0x8b, 0xca, 0x32, 0x93, 0x53, 0x9d,
  0x38, 0xa3, 0xd8, 0xa1, 0xaa, 0x93, 0xd8, 0xc0, 0x0a, 0x8d, 0x01, 0x01, 0x00, 0x00, 0xff, 0xff,
  0xbb, 0xd0, 0x0c, 0xc2, 0x45, 0x00, 0x00, 0x00,
}

// Reference imports to suppress errors if they are not otherwise used.
var _ context.Context
var _ grpc.ClientConn

// This is a compile-time assertion to ensure that this generated file
// is compatible with the grpc package it is being compiled against.
const _ = grpc.SupportPackageIsVersion4

// GreetServiceClient is the client API for GreetService service.
//
// For semantics around ctx use and closing/ending streaming RPCs, please refer to https://godoc.org/google.golang.org/grpc#ClientConn.NewStream.
type GreetServiceClient interface {
}

type greetServiceClient struct {
  cc *grpc.ClientConn
}

func NewGreetServiceClient(cc *grpc.ClientConn) GreetServiceClient {
  return &greetServiceClient{cc}
}

// GreetServiceServer is the server API for GreetService service.
type GreetServiceServer interface {
}

func RegisterGreetServiceServer(s *grpc.Server, srv GreetServiceServer) {
  s.RegisterService(&_GreetService_serviceDesc, srv)
}

var _GreetService_serviceDesc = grpc.ServiceDesc{
  ServiceName: "greet.GreetService",
  HandlerType: (*GreetServiceServer)(nil),
  Methods:     []grpc.MethodDesc{},
  Streams:     []grpc.StreamDesc{},
  Metadata:    "greet/greetpb/greet.proto",
}
```

---

## 16. Server Setup Boilerplate Code

### 16.1. gRPC Server Setup

* setup a gRPC Server with no service on it
* we'll see how to properly start & stop the Server on a defined **port**
* the point for this trial is just to be done with "boilerplate code"

Let's create another directory under `greet` as `greet_server`

* `pwd/greet/greet_server`

and add a file `server.go`

```go
package main

import "fmt"

func main() {
  fmt.Println("Hello world!")
}
```

make sure this code is running on your machine:

```bash
go run greet/greet_server/server.go
```

Now, you are ready to write a server: let's go back to the file:

```go
package main

import (
  "fmt"
  "log"
  "net"

  "../greetpb"      // can be vary depending on your choice

  "google.golang.org/grpc"
)

type server struct{}

func main() {
  fmt.Println("Hello world!")

  lis, err := net.Listen("tcp", "0.0.0.0:50051") // 50051 is a default port for gRPC
  if err != nil {
    log.Fatalf("Failed to listen: %v", err)
  }

  s := grpc.NewServer()
  greetpb.RegisterGreetServiceServer(s, &server{})

  if err := s.Serve(lis); err != nil {
    log.Fatalf("Failed to serve: %v", err)
  }
}
```

and you can run the code:

```bash
go run greet/greet_server/server.go
```

and see a pop up message for connection confirmation and your terminal will show "Hello world!" message.

---

## 17. Client Setup Boilerplate Code

### 17.1. gRPC Client Setup

* setup a gRPC Client that connects to our Server
* we'll see how to properly start & stop the Client
* the point for this trial is just to be done with "boilerplate code"

Let's create another directory under `greet` as `greet_client` and under that directory, create `client.go`

```go
package main

import (
  "fmt"
  "log"
  "../greetpb"
  "google.golang.org/grpc"
)

func main() {
  fmt.Println("Hello, I am a client.")

  cc, err := grpc.Dial("localhost:50051", grpc.WithInsecure())	// WithInsecure() for just now testing
  if err != nil {
    log.Fatalf("Could not connect: %v", err)
  }

  defer cc.Close()

  c := greetpb.NewGreetServiceClient(cc)
  fmt.Printf("Created client: %f", c)
}
```

* now you can run the client after running the `server.go` in the other terminal:

* terminal 1:

```bash
go run greet/greet_server/server.go
```

* terminal 2:

```bash
go run greet/greet_client/client.go
```

* be aware that since we don't pass any message, the client is actually runable without server running but this is basic flow how server and client work together.

---
