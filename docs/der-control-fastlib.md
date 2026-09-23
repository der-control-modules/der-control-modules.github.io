# der-control-fastlib Runtime

[Code & Installation Instructions](https://github.com/der-control-modules/der-control-fastlib){ .md-button }

der-control-fastlib is the lightweight runtime on which the Interoperability Framework and the DER control
applications are deployed. It is the DER Control Modules distribution of the Agent Energy Management System (AEMS)
library, a fork of [aems-lib-fastapi](https://github.com/VOLTTRON/aems-lib-fastapi) from the VOLTTRON team. It
replaces the full VOLTTRON platform with two small pieces:

* **A server** (`aems-server`) built on FastAPI that provides a WebSocket message bus for agent communication, a
  REST configuration store for agent settings, and health and version endpoints.
* **A client library** (`aems.client`) that provides an `Agent` base class with the same programming interface as a
  VOLTTRON agent: `vip.pubsub`, `vip.rpc`, `vip.config`, `RPC.export`, `Core.receiver`, and `Core.periodic`.

Each framework component and application runs as its own process, connects to the server over a WebSocket, and reads
its configuration from the server's configuration store. Because the client API mirrors VOLTTRON's, the same agent
code can be deployed on either runtime.

## Installation

Requirements: Python 3.10 or higher and pip.

```shell
python -m venv ~/der-control
source ~/der-control/bin/activate
pip install git+https://github.com/der-control-modules/der-control-fastlib
```

For development, clone the repository and use the bundled helper script, which provides Poetry-like commands for
pip (`install`, `install-prod`, `test`, `lint`, `format`, `build`, `check`):

```shell
git clone https://github.com/der-control-modules/der-control-fastlib
cd der-control-fastlib
./dev.py install
```

## Starting the server

```shell
export VOLTTRON_HOME=~/.der-control
aems-server --host 127.0.0.1 --port 8000
```

| Option            | Default                            | Description                                                                        |
|-------------------|------------------------------------|------------------------------------------------------------------------------------|
| `--host`          | `127.0.0.1`                        | Address to bind to. Use `0.0.0.0` to accept agents from other hosts.               |
| `--port`          | `8000`                             | Port to listen on.                                                                 |
| `--volttron-home` | `$VOLTTRON_HOME`                   | Base directory for runtime files.                                                  |
| `--config-dir`    | `$VOLTTRON_HOME/aems_config_store` | Directory in which the configuration store keeps agent configurations.             |

The server can also be started from Python with `aems.server.messagebus.start_server(host, port, config_store_dir)`.
Once running, interactive API documentation is served at `http://<host>:<port>/docs`.

## Server API

| Endpoint                                             | Purpose                                                                                     |
|------------------------------------------------------|---------------------------------------------------------------------------------------------|
| `WS /ws/{identity}`                                  | WebSocket connection for an agent with the given identity. Carries publish, subscribe, RPC, and RPC response messages. |
| `GET /config-store/list?agent_id=`                   | List stored configurations, optionally for one agent.                                       |
| `GET /config-store/{agent_id}/{config_name}`         | Read a configuration (`?raw=true` for the raw file).                                        |
| `PUT /config-store/{agent_id}/{config_name}`         | Store a configuration (JSON body, or text with the appropriate content type).               |
| `POST /config-store/{agent_id}/{config_name}/file`   | Upload a configuration file (form field `file`; `?config_type=` overrides the type).        |
| `DELETE /config-store/{agent_id}/{config_name}`      | Delete a configuration.                                                                     |
| `GET /health`, `GET /health/{agent_id}`              | Health status of all connected agents, or of one agent.                                     |
| `GET /version`                                       | Server version.                                                                             |

Agents subscribed to a configuration through `vip.config.subscribe` are notified when it is stored, updated, or
deleted, so configurations can be changed at runtime without restarting the agent.

## Running an agent

An agent is a Python class derived from `aems.client.agent.Agent`. The `run_agent` helper turns it into a command
line program that connects to the server:

```python
from aems.client.agent import Agent, Core, RPC, run_agent

class Listener(Agent):
    @Core.receiver("onstart")
    def _onstart(self, sender=None, **kwargs):
        self.vip.pubsub.subscribe("devices/", self._on_message)

    def _on_message(self, peer, sender, bus, topic, headers, message):
        print(topic, message)

    @RPC.export
    def get_version(self):
        return "1.0.0"

if __name__ == "__main__":
    raise SystemExit(run_agent(Listener))
```

```shell
python listener.py --identity listener --host 127.0.0.1 --port 8000 --config listener.json
```

| Argument / variable  | Description                                                                                          |
|----------------------|------------------------------------------------------------------------------------------------------|
| `--identity`         | The agent's identity on the bus (also the `agent_id` in the configuration store). Defaults to `AGENT_VIP_IDENTITY` or the lower-cased class name. |
| `--host`, `--port`   | Address of the `aems-server`.                                                                        |
| `--config`           | JSON or YAML configuration file. Its contents are stored in the configuration store as `config` on start-up. |
| `--volttron-home`    | Runtime directory, defaulting to `$VOLTTRON_HOME`.                                                   |

## Security

The server binds to `127.0.0.1` by default and does not implement authentication or TLS. For deployments in which
agents connect from other hosts, run the server behind a reverse proxy with TLS termination and restrict access to
the bus port at the network level.

## Releases

The repository uses GitHub Actions for continuous integration (tests on Python 3.10 to 3.12, formatting, linting,
security scans) and for releases. Pushing a tag of the form `v1.2.3-alpha.1` from the `develop` branch creates a
GitHub pre-release and notifies test servers; pushing a final tag `v1.2.3` from `main` creates a release and
publishes the package. Tags are created with `./dev.py version alpha|beta|rc|release|minor|major`.
