# gRPC

> gRPC is a modern, HTTP2-based protocol, that provides RPC semantics using the strongly-typed binary data format of Protocol Buffers across multiple languages (C++, C#, Golang, Java, Python, NodeJS, ObjectiveC, etc.)

[gRPC](https://grpc.io/) is a protocol for communication with services (it is a replacement for REST or GraphQL). gRPC uses [Protocol Buffers](https://developers.google.com/protocol-buffers) format to describe a service API. Protocol Buffers enables us to have a unified description of the API. It uses specific syntax (proto2, proto3) in \*.proto files.

## gRPC-web

> gRPC-Web is a cutting-edge spec that enables invoking gRPC services from modern browsers.

[gRPC-web](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-WEB.md) protocol is different from native gRPC protocol since there are several limitation in browser which makes direct use of gRPC protocol impossible. To be able to communicate with a service using native gRPC protocol, a proxy to translate between gRPC-web and gRPC protocols is needed. [Envoy](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/other_protocols/grpc) can act as the proxy.

### Implementations

To this date, there are two implementation of gRPC-web available via NPM.

- [grpc-web](https://www.npmjs.com/package/grpc-web)
- [improbabel-eng/grpc-web](https://www.npmjs.com/package/@improbable-eng/grpc-web)

Since [improbabel-eng/grpc-web](https://www.npmjs.com/package/@improbable-eng/grpc-web) has a better support for usage on both server and client (usefull if you need the same code to run in both browser and Node environment) and has overall better implementation, we use this package.

### NPM Dependencies

- **google-protobuf** - Protocol Buffers JavaScript code
- **grpc-tools** - NPM package for easy usage of `protoc` (protocol buffers compiler) binary
- **ts-protoc-gen** - a `protoc` plugin for compilation of \*.proto files to JavaScript code that uses `@improbable-eng/grpc-web` and supports TypeScript types
- **@improbable-eng/grpc-web** - browser JavaScript implementation for gRPC-web calls
- **@improbabel-eng/grpc-web-node-http-transport** - a transport layer to enable `@improbable-eng/grpc-web` working in Node environment to reuse the same code in the client and on the server

### Usage

Say you have a description of a service in `./src/proto` folder. You want these \*.proto files to be compiled to JavaScript code you can use in your application. That is what afore mentioned `protoc` compiler does. It compiles \*.proto files to \*.js files. For easy usage with NPM, we use `grpc-tools` which gives us a binary for `prococ`

`protoc` alone would compile just gRPC messages in JavaScript since it is plain JavaScript. Communication layer is added by `improbabel-eng/grpc-web`. To use it, you need to tell it to the compiler somehow. That is what is `ts-protoc-gen` plugin for.

Now, you can write an NPM script to generate JavaScript code:

```
{
    "grpc": "grpc_tools_node_protoc --plugin=ts-protoc-gen=./node_modules/.bin/ts-protoc-gen  --js_out=import_style=commonjs,binary:./src/build  --ts_out="service=grpc-web:./src/build -I ./src/proto ./src/proto/**/*.proto"
}
```

Running this script would generate JavaScript code to `./src/build` folder from \*.proto files in the `./src/proto` folder.

#### Transport layers

As mentioned before, the `improbable-eng/grpc-web` implemtantion of gRPC-web protocol has several advantages over `grpc-web` and transport layers are one of them.

`improbable-eng/grpc-web` supports several browser based layers: **cross-browser** (fetch with fallback to xhr), **fetch**, **xhr**, **websockets**. These are to be used in the browser environment. If you need to use gRPC-web in the Node environment (for example in SSR apps), you need a special transport layer `@improbabel-eng/grpc-web-node-http-transport`

```
import { grpc } from '@improbable-eng/grpc-web';
import { NodeHttpTransport } from '@improbable-eng/grpc-web-node-http-transport';

grpc.setDefaultTransport(NodeHttpTransport());
```

#### Example - list all articles from the article service:

```
import { ListArticlesRequest } from 'common/grpc/v1/article_pb';
import { ArticleClient } from 'common/grpc/v1/article_pb_service';

const serviceUrl = 'https://...`;

const articleClient = new ArticleClient(serviceUrl);

const requestMessage = new ListArticlesRequest();

const response = articleClient.listArticles(requestMessage, (error, responseMessage) => {
    console.log(error, responseMessage);
})
```
