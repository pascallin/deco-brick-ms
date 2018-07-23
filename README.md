# deco-brick-ms
rpc micro-service framework for typescript

## Concept
Only `grpc` is avaliable now.

There are `GrpcServer`，`GrpcClient`，`IBrickService`，`EtcdDiscovery`  for useage.

## GrpcServer
### server config 
```
{
  host: HOST, // optional, default is '0.0.0.0'
  port: PORT,
  protoPath: PROTOPATH,
}
```
### server usage

The protobuf file example as below:
```
syntax = "proto3";
package test;
service Test {
	rpc check (req) returns (res) {}
}
message req {
	string data = 1;
}
message res {
	string status = 1;
}
```
Create a server
```
import { GrpcServer, IBrickService } from "deco-brick-rpc";

class TestService implements IBrickService {
  public name: string =  "Test";
  public async check(params: any): Promise<object> {
    return { status: "success" };
  }
}

const test = new GrpcServer({
  port: 50051,
  protoPath: __dirname + "/test.proto",
});
test.setServices([ TestService ]);
test.start();
```

## GrpcClient
Client share the same protobuf file as server.

### client config
```
{
  protoPath: PROTOPATH,
  host: HOST, // optional, default is 'localhost'
  port: PORT
}
```
### client usage
Here is a client demo
```
import { GrpcClient } from "deco-brick-rpc";
const log = console.log;

const rpc = new GrpcClient({
  port: 50051,
  protoPath: __dirname + "/test.proto",
});

log("package name: ", rpc.packageName);

rpc.client.Test.check().sendMessage({data: "you"}).then((data: any) => {
  log(data);
}).catch((e: any) => {
  log(e.message);
});
```

## IBrickService
The service interface is for standardization.
```
interface IBrickService {
  name: string;
}
```

## EtcdDiscovery
You can use etcd as a discovery centre for multiple application.

### discovery config
```
{
  namespace: NAMESPACE,
  url: ETCD_URL // e.g. "localhost:2379"
}
```

### server usage
```
import { EtcdDiscovery, GrpcServer, IBrickService } from "deco-brick-rpc";

class TestService implements IBrickService {
  public name: string =  "Test";
  public async check(params: any): Promise<object> {
    return { status: "server 1 success" };
  }
}

const test = new GrpcServer({
  discovery: new EtcdDiscovery({
    namespace: "deco",
    url: "localhost:2379",
  }),
  port: 50051,
  protoPath: __dirname + "/../test.proto",
});
test.setServices([ TestService ]);
test.start();
```

### client usage
```
import { EtcdDiscovery, GrpcClient } from "deco-brick-rpc";
const log = console.log;

const discovery = new EtcdDiscovery({
  namespace: "deco",
  url: "localhost:2379",
});
const { host, port } = discovery.discover("test");
log(host, port);
const rpc = new GrpcClient({
  host,
  port,
  protoPath: __dirname + "/../test.proto",
});
rpc.client.Test.check().sendMessage({data: "you"}).then((data: any) => {
  log(data);
}).catch((e: any) => {
  log(e.message);
});

```

# examples
- [simple](https://github.com/pascallin/deco-brick-rpc/tree/dev/src/example/simple)
- [multiple with etcd](https://github.com/pascallin/deco-brick-rpc/tree/dev/src/example/multiple)

# TODO
- thrift support