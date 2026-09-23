# Deploying the Framework and Applications

This page walks through a complete deployment of the [Interoperability Framework](interoperability-framework.md) and
the [Applications](rt-control.md) on the [der-control-fastlib runtime](der-control-fastlib.md). Every component is a
separate process that connects to one `aems-server`, so a deployment consists of starting the server, starting the
framework components that talk to devices, and then starting the applications that consume the normalized data.

!!! note "Deploying on VOLTTRON"
    The components can also be deployed on a full Eclipse VOLTTRON platform. In that case, replace the `run_agent`
    commands below with the `vctl install` and `vctl config store` commands given on each component's page.

## Deployment layout

| Layer                | Component                                                              | Identity                  | Purpose                                                              |
|----------------------|------------------------------------------------------------------------|---------------------------|----------------------------------------------------------------------|
| Runtime              | [der-control-fastlib](der-control-fastlib.md) `aems-server`             |                           | Message bus and configuration store.                                 |
| Framework            | [Interoperability Service](interoperability-service.md)                 | `platform.presentation`   | Resolves identifiers and transforms data models.                     |
| Framework            | [Message Bus Adapter](message-bus-adapter.md) with [MQTT](mqtt.md) or [NATS](nats.md) proxy | `platform.bus_adapter` | Relays data to and from OpenFMB buses.                          |
| Framework            | Device drivers                                                         | `platform.driver`         | Direct communication with SunSpec Modbus, DNP3, and IEEE 2030.5 devices. |
| Application          | [Grid Signals](grid-signals.md)                                        | `grid.signals`            | Price and CO₂ signals.                                               |
| Application          | [Forecaster Agent](forecaster-agent.md)                                | `load.forecaster`         | Building load forecast.                                              |
| Application          | [Scheduler](scheduler.md)                                              | `agent.scheduler`         | Day-ahead dispatch.                                                  |
| Application          | [Real-Time Control Agent](rt-control.md)                               | `der.rtcontrol`           | Real-time actuation with MESA and novel modes.                       |

Start the layers in this order. The framework components must be connected before the applications start, because
the applications resolve their device topics through the Interoperability Service.

## 1. Prepare the host

```shell
sudo apt-get install -y python3.10 python3.10-venv git
python3.10 -m venv ~/der-control
source ~/der-control/bin/activate
pip install --upgrade pip
export VOLTTRON_HOME=~/.der-control
mkdir -p $VOLTTRON_HOME/configs
```

Add the `VOLTTRON_HOME` export to the shell profile so that every component finds the same runtime directory.

## 2. Install the runtime and start the server

```shell
pip install git+https://github.com/der-control-modules/der-control-fastlib
aems-server --host 127.0.0.1 --port 8000
```

Leave the server running in its own terminal, or install it as a service (see [Running as services](#running-as-services)).
Confirm it is up:

```shell
curl http://127.0.0.1:8000/version
curl http://127.0.0.1:8000/health
```

## 3. Deploy the framework

Install the framework packages into the same virtual environment:

```shell
pip install git+https://github.com/der-control-modules/interoperability-service
pip install git+https://github.com/der-control-modules/lib-protocol-proxy-mqtt   # and/or lib-protocol-proxy-nats
pip install git+https://github.com/der-control-modules/message-bus-adapter
```

### Interoperability Service

Write the mappings for your devices to `$VOLTTRON_HOME/configs/interoperability.json` (see the
[configuration example](interoperability-service.md#configuration)). Transforms between IEC 61850-7-420,
IEEE 1815.2, and SunSpec are [bundled with the service](interoperability-service.md#bundled-transforms) and loaded
automatically, so `transforms` only needs entries for site-specific formats. Then start the service:

```shell
interoperability-service --identity platform.presentation --host 127.0.0.1 --port 8000 \
    --config $VOLTTRON_HOME/configs/interoperability.json
```

Mappings and transforms can be added later without a restart, either by publishing to the `mapper/update` topic or
by storing an updated configuration:

```shell
curl -X PUT http://127.0.0.1:8000/config-store/platform.presentation/config \
     -H "Content-Type: application/json" --data @$VOLTTRON_HOME/configs/interoperability.json
```

### Message Bus Adapter

If the deployment exchanges data with an OpenFMB bus, write the adapter configuration
(see [Message Bus Adapters](message-bus-adapter.md#configuration)) and start the adapter. It launches the MQTT or
NATS proxy process for each configured broker:

```shell
python -m bus_adapter.agent --identity platform.bus_adapter --host 127.0.0.1 --port 8000 \
    --config $VOLTTRON_HOME/configs/bus_adapter.json
```

### Device drivers

Devices reached directly over SunSpec Modbus, DNP3 (IEEE 1815.2), or IEEE 2030.5 are served by a driver process
that publishes device data on `devices/<campus>/<building>/<device>/all` and accepts set points over RPC. Register
each device's publication and RPC topics as canonical resources in the Interoperability Service configuration so
that applications can address them by identifier rather than by protocol.

## 4. Deploy the applications

Install the application packages:

```shell
pip install git+https://github.com/der-control-modules/grid-signals
pip install git+https://github.com/der-control-modules/load-forecaster
pip install git+https://github.com/der-control-modules/scheduler
pip install git+https://github.com/der-control-modules/realtime-control-agent
```

Write one configuration file per application in `$VOLTTRON_HOME/configs/`, using the examples on the
[Grid Signals](grid-signals.md#configuration), [Forecaster Agent](forecaster-agent.md#configuration),
[Scheduler](scheduler.md#configuration), and [Real-Time Control Agent](rt-control.md#configuration) pages. Then start
the applications, inputs first:

```shell
python -m grid_signals_agent.agent --identity grid.signals   --host 127.0.0.1 --port 8000 \
    --config $VOLTTRON_HOME/configs/grid_signals.json
python -m load_forecaster.agent    --identity load.forecaster --host 127.0.0.1 --port 8000 \
    --config $VOLTTRON_HOME/configs/forecaster.json
python -m scheduler.agent          --identity agent.scheduler --host 127.0.0.1 --port 8000 \
    --config $VOLTTRON_HOME/configs/scheduler.json
python -m rt_control.agent         --identity der.rtcontrol   --host 127.0.0.1 --port 8000 \
    --config $VOLTTRON_HOME/configs/rt_control.json
```

The Scheduler's `forecast_config` should point at the topics published by Grid Signals
(`devices/<campus>/grid_information/price/all`, point `tou`) and the Forecaster
(`devices/<campus>/<building>/forecast/all`, point `load`), and the Real-Time Control Agent's `ess` block should
name the device topics registered in the Interoperability Service.

### Novel control modes

If the Real-Time Control Agent is configured with [novel modes](novel-real-time-control.md), the host additionally
needs Julia and the Control Evaluation Engine, and the virtual environment must use a dynamically linked Python.
Follow [ES Control Integration](es-control-integration.md#installation) before starting the agent, and set
`ctrl_eval_engine_app_path` and `julia_path` in its configuration.

## 5. Verify the deployment

```shell
curl http://127.0.0.1:8000/health                       # every identity above should report healthy
curl "http://127.0.0.1:8000/config-store/list"           # one 'config' entry per component
```

Run a listener (see [Running an agent](der-control-fastlib.md#running-an-agent)) subscribed to `devices/` and
`record/` to watch device data, grid signals, forecasts, and the published schedule flow across the bus.

## Running as services

For an unattended deployment, run each process under systemd. A unit for the server:

```ini
# /etc/systemd/system/aems-server.service
[Unit]
Description=der-control-fastlib message bus
After=network.target

[Service]
User=volttron
Environment=VOLTTRON_HOME=/home/volttron/.der-control
ExecStart=/home/volttron/der-control/bin/aems-server --host 127.0.0.1 --port 8000
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

and a template for the components, one instance per identity:

```ini
# /etc/systemd/system/der-control@.service
[Unit]
Description=DER control component %i
After=aems-server.service
Requires=aems-server.service

[Service]
User=volttron
Environment=VOLTTRON_HOME=/home/volttron/.der-control
EnvironmentFile=/home/volttron/.der-control/env/%i
ExecStart=/home/volttron/der-control/bin/python -m ${MODULE} --identity %i --host 127.0.0.1 --port 8000 \
    --config /home/volttron/.der-control/configs/%i.json
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

where each environment file (for example `env/der.rtcontrol`) sets `MODULE=rt_control.agent`. Enable the services
in dependency order:

```shell
sudo systemctl enable --now aems-server
sudo systemctl enable --now der-control@platform.presentation der-control@platform.bus_adapter
sudo systemctl enable --now der-control@grid.signals der-control@load.forecaster \
                            der-control@agent.scheduler der-control@der.rtcontrol
```

## Updating a deployment

Upgrade a component by reinstalling its package and restarting its service; configurations in the configuration store
are preserved across restarts:

```shell
pip install --upgrade git+https://github.com/der-control-modules/realtime-control-agent
sudo systemctl restart der-control@der.rtcontrol
```

Configuration changes never require a restart: store the new configuration with a `PUT` to the configuration store
and the component receives an update notification.
