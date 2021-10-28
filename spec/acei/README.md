---
order: 1
parent:
  title: ACEI
  order: 2
---

# ACEI

ACEI stands for "**A**pplication **C**onsensus **E**ngine **I**nterface".
ACEI is the interface between a consensus engine (state-machine replication engine)
and your application (the actual state machine). It consists of a set of
_methods_, each with a corresponding `Request` and `Response`message type.
To perform state-machine replication, a consensus engine calls the ACEI methods on the
ACEI application by sending the `Request*` messages and receiving the `Response*` messages in return.

All ACEI messages and methods are defined in [protocol buffers](https://github.com/daotl/acei/blob/master/proto/daotl/acei/types.proto).
This allows a consensus engine to run with applications written in many programming languages.

This specification is split as follows:

- [Methods and Types](./acei.md) - complete details on all ACEI methods and
  message types
- [Applications](./apps.md) - how to manage ACEI application state and other
  details about building ACEI applications
- [Client and Server](./client-server.md) - for those looking to implement their
  own ACEI application servers
