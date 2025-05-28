[Code & Installation Instructions](https://github.com/der-control-modules/interoperability-agent){ .md-button }

The VOLTTRON interoperability agent provides a standard, protocol-agnostic interface for
Distributed Energy Resources (DER) devices by mapping IEC 61850-7-420 data models to protocols such as SunSpec Modbus,
IEEE 1815.2, IEEE 2030.5, and custom protocols, in line with IEEE 1547 standards.
It translates the required functionalities into specific protocols for device control.
As illustrated [](#interop-one-to-many), an actor wishing to communicate with devices using multiple protocols
compliant with IEEE 1547 can use the interoperability agent to manage all such devices. 

Figure: The interoperability agent provides a standard, protocol-agnostic interface to DER devices.
It maps IEC 61850-7-420 data models to the appropriate point names in protocols that implement specific
versions of these models, in accordance with the IEEE 1547 interoperability standard, including SunSpec Modbus,
IEEE 1815.2, and IEEE 2030.5 {#interop-one-to-many}

![](images/interop-one-to-many.png)


The actor issues commands using IEC 61850-7-420 names, which the interoperability agent maps to specific protocols,
such as SunSpec Modbus, MESA-DER (DNP3), or IEEE 2030.5. The corresponding points are then commanded on the devices
via the VOLTTRON Platform Driver. If a device does not implement the data models specified in IEEE 1547,
or the necessary functionality, the Interoperability Agent will lack a method for controlling it.
In such cases, another agent may be used to implement the required functionality and expose a compliant data model.
This secondary agent serves as an intermediary between the Interoperability Agent and the Platform Driver,
as illustrated in [](#interop-direct-and-non-compliant).

Figure: The Interoperability agent can directly execute IEEE 1547 methods on devices
with native support or alongside additional control agents to apply IEEE 1547 methods
to devices lacking native implementation.
{#interop-direct-and-non-compliant}

![](images/interop-direct-and-non-compliant.png)
