# Play-Protobuf

This plugin allows you to use protocol buffers as arguments to your controllers' methods and to return protocol buffers to the client.

It is protocol-compatible with [ProtoRPC](http://code.google.com/appengine/docs/python/tools/protorpc/overview.html), and makes writing RPC services super easy. For instance you could have a **myproto.proto** file declaring a few messages like this:

```proto
// Let's be compatible with mobile phones
option optimize_for = LITE_RUNTIME;

// Generated messages will be inner classes of com.github.jcayzac.Protos
option java_package = "com.github.jcayzac";
option java_outer_classname = "Protos";

// Protobuf package name
package protos;

message MyRequest {
    required string name = 1;
    optional int32  age  = 2;
}

message MyResponse {
    required string value = 1;
}
```

Then, using the classes generated by `protoc` in your controllers becomes as simple as:

```java
package controllers;
import play.mvc.Controller;
import com.github.jcayzac.Protos;

public class MyRpc extends Controller {

    public static void myRPCMethod(final Protos.MyRequest request) {
        Protos.MyResponse.Builder response = Protos.MyResponse.newBuilder();
        response.setValue(doSomething(request.getName(), request.getAge()));

        // controllers.PBActions is provided by Play-Protobuf.
        PBActions.render(response.build());
    }
}
```

You can test it from the command line, using **curl** and **protoc**:

```sh
$ echo "name: 'Bob X'" \
| protoc --encode=protos.MyRequest myproto.proto \
| curl -H 'content-type:application/x-google-protobuf' --data-binary @- http://localhost:9000/your/controller/route \
| protoc --decode=protos.MyResponse myproto.proto
value: 'Hello, Bob X!'
```

## Important notes

* The RPC methods must have one argument, and one only.
* The RPC methods' single argument should always be named **request**. Otherwise the magic won't work.
* The requests must have their content-type set to **application/x-google-protobuf**. Responses rendered with **PBActions.render()** will be served with that MIME type, too.

## Known issues or shortcomings

* No support for JSON serialization of protocol buffers (which ProtoRPC supports, for debugging purpose)

## License

This code is released under the terms of the Simplified BSD License below.

```
Copyright 2011 Julien Cayzac. All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

   1. Redistributions of source code must retain the above copyright notice,
      this list of conditions and the following disclaimer.

   2. Redistributions in binary form must reproduce the above copyright
      notice, this list of conditions and the following disclaimer in the
      documentation and/or other materials provided with the distribution.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS "AS IS" AND ANY EXPRESS
OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES
OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN
NO EVENT SHALL THE COPYRIGHT HOLDERS OR CONTRIBUTORS BE LIABLE FOR ANY
DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
(INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF
THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

The views and conclusions contained in the software and documentation are
those of the authors and should not be interpreted as representing official
policies, either expressed or implied, of the copyright holders.
```

