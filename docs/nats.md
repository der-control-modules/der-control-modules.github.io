# NATS Protocol Proxy

[Code & Installation Instructions](https://github.com/der-control-modules/lib-protocol-proxy-nats){ .md-button }

The NATS Protocol Proxy connects the [Message Bus Adapter](message-bus-adapter.md) to a NATS server. NATS is a
lightweight, high-performance publish/subscribe messaging system used as a transport in OpenFMB deployments; see
[OpenFMB Integration](openfmb.md).

The proxy is implemented in the `protocol-proxy-nats` Python package on top of the `protocol-proxy` library and the
[nats-py](https://pypi.org/project/nats-py/) client. Like the [MQTT proxy](mqtt.md), it runs as a separate process
launched and supervised by the Message Bus Adapter's proxy manager. Because NATS provides an asynchronous client, the
NATS proxy is built on the asyncio variant of the protocol proxy base class.

## Behavior

* On start-up the proxy connects to the configured NATS server(s) and registers itself with the adapter.
* When the adapter requests a subscription, the proxy subscribes to the requested NATS subjects.
* Each message received from a subscribed subject is forwarded to the adapter with its subject, payload, and NATS
  headers. The adapter resolves the subject through the [Interoperability Service](interoperability-service.md) and
  publishes the transformed payload on the local bus.
* NATS subjects use `.` as the segment delimiter, which the adapter uses when resolving subjects to Uniquely
  Addressable Identifiers.

## Configuration

The proxy is configured through the `adapters` entries of the Message Bus Adapter configuration when `bus_type` is
`nats`:

| Parameter | Required | Type           | Description                                                                                 |
|-----------|----------|----------------|---------------------------------------------------------------------------------------------|
| `servers` | true     | string or list | One or more NATS server URLs, for example `nats://nats.example.org:4222`.                   |

Additional keyword arguments accepted by the nats-py `connect` call (for example TLS settings) are passed through to
the client. The `--servers` option is accepted on the command line when the proxy is launched directly for testing.

## Installation

```shell
pip install git+https://github.com/der-control-modules/lib-protocol-proxy-nats
```

Install the package into the same virtual environment as the [Message Bus Adapter](message-bus-adapter.md). The
adapter discovers the proxy automatically when `bus_type` is set to `nats`.
