---
title: "The speed of gRPC Python is very impressive and that deserves a dive"
date: 2023-07-14T15:43:15-07:00
draft: false
tags: ["grpc", "python"]
---

# Intro

Recently, I did a small experiment to compare the [performance](https://github.com/aspcompiler/grpc-perf) of gRPC Python and other languages. The result is very impressive. I would expect to see a interpreted code many times slower than a native code. Intuitively, this can only happen if the code is mostly native. Let us dive in to see how it was implemented.

# gRPC Python implementation

gRPC has 2 major parts. Its serializer is built upon [protobuf](https://github.com/protocolbuffers/protobuf). The RPC part is built upon HTTP/2.

The gRPC project is driven by the RFP process. gRPC Python has the following RFPs:

* L13 https://github.com/grpc/proposal/blob/master/L13-python-interceptors.md 

* L42 https://github.com/grpc/proposal/blob/master/L42-python-metadata-flags.md 

* L44 https://github.com/grpc/proposal/blob/master/L44-python-rich-status.md 

* L46 https://github.com/grpc/proposal/blob/master/L46-python-compression-api.md 

* L54 https://github.com/grpc/proposal/blob/master/L54-python-server-wait.md 

* L58 https://github.com/grpc/proposal/blob/master/L58-python-async-api.md 

* L64 https://github.com/grpc/proposal/blob/master/L64-python-runtime-proto-parsing.md 

* L65 https://github.com/grpc/proposal/blob/master/L65-python-package-name.md 

* L78 https://github.com/grpc/proposal/blob/master/L78-python-rich-server-context.md 

* L95 https://github.com/grpc/proposal/blob/master/L95-python-reflection-client.md 

Out of them, L58 and L64 are the most interesting one. L58 is the async API. L64 is the runtime proto parsing. They are the core of the implementation.

# Async API
 
L58 is a great document that is fun to read.

The most interesting part of `grpcio`, the async Python gRPC API, is that it is a wrapper over the gRPC C++ API but used the Python async model. It other words, it does not use the Python networking stack but used the networking stack of the C++ API. How are they married together?

It turns out that the C++ API has a [CompletionQueue](https://grpc.github.io/grpc/cpp/classgrpc_1_1_completion_queue.html) interface. It allow
the clients to [poll for the completion status with tag](https://grpc.io/docs/languages/cpp/async/).

Any async framework requires a mechanism to yield control. Python uses [generator to yield](https://stackoverflow.com/questions/49005651/how-does-asyncio-actually-work). Therefore, we can imaging that the wrapper just need to poll the completion queue and yield the results in the generator.

The result is a very elegant API that allows users to implement a streaming server with extreme simplicity:

```python
class Greeter(helloworld_pb2_grpc.GreeterServicer):
    async def StreamingHi(self, request_iterator, context):
        async for request in request_iterator:
            yield response

```

`asyncio` is integrated C++ API using [Cython](https://cython.org/).

# Cython

I was previously not familiar with Cython. I thought it is just a Python to C++ compiler. Many of the integrations that I have seen
where implemented using [pybind11](https://github.com/pybind/pybind11) for Python/C++ binding or [pyo3](https://github.com/PyO3/pyo3)
for Python/Rust binding.

It turns out that Cython is also very interesting. It is a very old project dating back to 15 years ago. It has 2 syntaxes, one with the .pyx extension and the other with the .pxd extension. 

The .pyx extension is a superset of Python. It allows us to add type hints to Python so that it can generate good C++ code; remember that Python
did not have type hints 15 years ago so that .pyx extension was invented to define the Python interface.

The .pxd extension allows us to export C++ API to Python.

The 2 extensions are sometimes used together to allow Python and C++ to meet half way. The tool then generates C++ code that can be compiled into a Python extension. We can inspect the generated code.

# Protobuf

`asyncio` uses the same Protobuf code as the sync code. This implementation uses the C++ protobuf runtime already built into the grpcio-tools C extension to parse the protocol buffers and generate textual Python code in memory. This code is then used to instantiate the modules to be provided to the calling application. This will be more clear when we look at the generated files.

# Generated files

The code generated from the proto file looks like:

* server_streaming_pb2.pyi	
* server_streaming_pb2.py		
* server_streaming_pb2_grpc.py

The first 2 files are for protobuf. The last one is for gRPC.

`server_streaming_pb2.pyi` looks like:

```python
class StreamingFromServerRequest(_message.Message):
    __slots__ = ["num_bytes"]
    NUM_BYTES_FIELD_NUMBER: _ClassVar[int]
    num_bytes: int
    def __init__(self, num_bytes: _Optional[int] = ...) -> None: ...

class StreamingFromServerResponse(_message.Message):
    __slots__ = ["data"]
    DATA_FIELD_NUMBER: _ClassVar[int]
    data: bytes
    def __init__(self, data: _Optional[bytes] = ...) -> None: ...
```

Note that it just define the interface for Python tools to use. It does not contain any implementation. The implementation is in 
` server_streaming_pb2.py`:

```python
DESCRIPTOR = _descriptor_pool.Default().AddSerializedFile(b'\n\x16server_streaming.proto\x12\x10server_streaming\"/\n\x1aStreamingFromServerRequest\x12\x11\n\tnum_bytes\x18\x01 \x01(\x05\"+\n\x1bStreamingFromServerResponse\x12\x0c\n\x04\x64\x61ta\x18\x01 \x01(\x0c\x32\x87\x01\n\x0fServerStreaming\x12t\n\x13StreamingFromServer\x12,.server_streaming.StreamingFromServerRequest\x1a-.server_streaming.StreamingFromServerResponse0\x01\x62\x06proto3')

_globals = globals()
_builder.BuildMessageAndEnumDescriptors(DESCRIPTOR, _globals)
_builder.BuildTopDescriptorsAndMessages(DESCRIPTOR, 'server_streaming_pb2', _globals)
if _descriptor._USE_C_DESCRIPTORS == False:

  DESCRIPTOR._options = None
  _globals['_STREAMINGFROMSERVERREQUEST']._serialized_start=44
  _globals['_STREAMINGFROMSERVERREQUEST']._serialized_end=91
  _globals['_STREAMINGFROMSERVERRESPONSE']._serialized_start=93
  _globals['_STREAMINGFROMSERVERRESPONSE']._serialized_end=136
  _globals['_SERVERSTREAMING']._serialized_start=139
  _globals['_SERVERSTREAMING']._serialized_end=274
```

This is not quite readable, but we can imagine that it just deserialize some data and load into the C++ code. The C++ code
then have the data structures needed to parse the protocol buffers.

Let us now look at the gRPC code `server_streaming_pb2_grpc.py`:

```python
class ServerStreaming(object):
    """Missing associated documentation comment in .proto file."""

    @staticmethod
    def StreamingFromServer(request,
            target,
            options=(),
            channel_credentials=None,
            call_credentials=None,
            insecure=False,
            compression=None,
            wait_for_ready=None,
            timeout=None,
            metadata=None):
        return grpc.experimental.unary_stream(request, target, '/server_streaming.ServerStreaming/StreamingFromServer',
            server__streaming__pb2.StreamingFromServerRequest.SerializeToString,
            server__streaming__pb2.StreamingFromServerResponse.FromString,
            options, channel_credentials,
            insecure, call_credentials, compression, wait_for_ready, timeout, metadata)
```

As we can see, it calls `grpc.experimental.unary_stream` and passes the serializer/deserializer from the protobuf code. The `unary_stream` is because this method happen to be a unary stream call. Other possible calls unary unary, stream unary, and stream stream.

Isn't this implementation very elegant? 

# Some more details

It turns out there are several C++ protobuf implementations. We can find out which one is used by running the following code:

```python
$ python 
Python 3.10.9 (main, Dec 7 2022, 13:47:07) [GCC 12.2.0] on linux 
Type "help", "copyright", "credits" or "license" for more information. 
>>> from google.protobuf.internal import api_implementation 
>>> print(api_implementation.Type()) 
upb 

``` 

Upc is currently Preferred. We can find more details from https://github.com/protocolbuffers/upb/tree/main/python 

# Conclusion

gRPC Python is extremely fast because it is mostly built around the C++ API.

The implementation using Cython is very elegant and we can borrow the idea to speed up our own Python code.
