# [Protocol Details](https://github.com/open-telemetry/opentelemetry-proto/blob/main/docs/specification.md#protocol-details)

OTLP defines the encoding of telemetry data and the protocol used to exchange data between the client and the server.

This specification defines how OTLP is implemented over [gRPC](https://grpc.io/) and HTTP transports and specifies [Protocol Buffers schema](https://developers.google.com/protocol-buffers/docs/overview) that is used for the payloads.

OTLP is a request/response style protocol: the clients send requests, and the server replies with corresponding responses. This document defines one request and response type: `Export`.

All server components MUST support the following transport compression options:

- No compression, denoted by `none`.
- Gzip compression, denoted by `gzip`.
