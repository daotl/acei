---
order: 3
title: Client and Server
---

# Client and Server

This section is for those looking to implement their own ACEI Server, perhaps in
a new programming language.

You are expected to have read [ACEI Methods and Types](./acei.md) and [ACEI
Applications](./apps.md).

## Message Protocol

The message protocol consists of pairs of requests and responses defined in the
[protobuf file](../../proto/daotl/acei/types.proto).

Some messages have no fields, while others may include byte-arrays, strings, integers,
or custom protobuf types.

For more details on protobuf, see the [documentation](https://developers.google.com/protocol-buffers/docs/overview).

For each request, a server should respond with the corresponding
response, where the order of requests is preserved in the order of
responses.

## Server Implementations

To use ACEI in your programming language of choice, there must be a ACEI
server in that language. Tendermint supports three implementations of the ACEI, written in Go:

- In-process ([Golang](https://github.com/daotl/go-acei))
- ACEI-socket
- GRPC

The latter two can be tested using the `abci-cli` by setting the `--abci` flag
appropriately (ie. to `socket` or `grpc`).

See examples, in various stages of maintenance, in
[Go](https://github.com/daotl/go-acei/tree/master/server),

### In Process

The simplest implementation uses function calls within Golang.
This means ACEI applications written in Golang can be compiled with Tendermint Core and run as a single binary.

### GRPC

If GRPC is available in your language, this is the easiest approach,
though it will have significant performance overhead.

To get started with GRPC, copy in the [protobuf
file](../../proto/daotl/acei/types.proto)
and compile it using the GRPC plugin for your language. For instance,
for golang, the command is `protoc --go_out=plugins=grpc:. types.proto`.
See the [grpc documentation for more details](http://www.grpc.io/docs/).
`protoc` will autogenerate all the necessary code for ACEI client and
server in your language, including whatever interface your application
must satisfy to be used by the ACEI server for handling requests.

Note the length-prefixing used in the socket implementation (TSP) does not apply for GRPC.

### TSP

Tendermint Socket Protocol is an asynchronous, raw socket server which provides ordered message passing over unix or tcp.
Messages are serialized using Protobuf3 and length-prefixed with a [signed Varint](https://developers.google.com/protocol-buffers/docs/encoding?csw=1#signed-integers)

If GRPC is not available in your language, or you require higher
performance, or otherwise enjoy programming, you may implement your own
ACEI server using the Tendermint Socket Protocol. The first step is still to auto-generate the relevant data
types and codec in your language using `protoc`. In addition to being proto3 encoded, messages coming over
the socket are length-prefixed to facilitate use as a streaming protocol. proto3 doesn't have an
official length-prefix standard, so we use our own. The first byte in
the prefix represents the length of the Big Endian encoded length. The
remaining bytes in the prefix are the Big Endian encoded length.

For example, if the proto3 encoded ACEI message is 0xDEADBEEF (4
bytes), the length-prefixed message is 0x0104DEADBEEF. If the proto3
encoded ACEI message is 65535 bytes long, the length-prefixed message
would be like 0x02FFFF....

The benefit of using this `varint` encoding over the old version (where integers were encoded as `<len of len><big endian len>` is that
it is the standard way to encode integers in Protobuf. It is also generally shorter.

As noted above, this prefixing does not apply for GRPC.

An ACEI server must also be able to support multiple connections, as
Tendermint uses four connections.

### Async vs Sync

The main ACEI server (ie. non-GRPC) provides ordered asynchronous messages.
This is useful for DeliverTx and CheckTx, since it allows Tendermint to forward
transactions to the app before it's finished processing previous ones.

Thus, DeliverTx and CheckTx messages are sent asynchronously, while all other
messages are sent synchronously.

## Client

There are currently two use-cases for an ACEI client. One is a testing
tool, as in the `abci-cli`, which allows ACEI requests to be sent via
command line. The other is a consensus engine, such as Tendermint Core,
which makes requests to the application every time a new transaction is
received or a block is committed.

It is unlikely that you will need to implement a client. For details of
our client, see
[here](https://github.com/daotl/go-acei/tree/master/client).
