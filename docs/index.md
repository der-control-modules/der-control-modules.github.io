# Introduction

Energy storage systems (ESS) is the most capable resource, supporting grid stability, enhancing flexibility, and adding
new services to the electrical system. However, the integration of ESS can be challenging and costly due to evolving communication standards
and operating modes. In order to reduce integration complexity and avoid interoperability issues that delay seamless deployment,
this work provides an open-source interoperable communication and control framework for ESS. 
This framework provides a protocol-agnostic interface for ESS by mapping the data models
of IEC 61850-7-420 to protocols according to IEEE 1547 standards. In addition, controls are developed based on
existing MESA modes and a control framework that includes scheduling and real-time control mechanisms
to provide grid services. This framework streamlines the integration of ESS by translating communication standards, 
providing examples of complex control modes, and therefore reducing complexity, risk, and cost associated with full utilization of ESS capabilities across diverse vendors. 

# Background

ESS improves energy efficiency, reduces electricity costs,
and improves the reliability of the grid. It can provide critical services like frequency regulation
and voltage control, helping maintain grid stability and prevent blackouts. As more distributed and intermittent
energy sources, such as photovoltaic (PV) and wind power, along with bidirectional components like electric
vehicles (EVs), are integrated into the grid, the role of ESS is expanding. Advanced control and optimization
algorithms are driving research into ESS management. For example, the financial advantages of peak shaving
and energy arbitrage for BTM ESS under demand charge tariffs, while other works examined ESS integration to
mitigate the short-term variability of renewable energy, further evaluated various energy storage technologies
for grid services. 

Interoperability ensures that various energy storage systems from different manufacturers can communicate
and coordinate charging/discharging activities efficiently. Several communication protocols and standards
have been developed to address Sunspc Modbus, Distributed Network Protocol 3 (DNP3) and global standards like
IEC 61850 for power utility operation. IEEE 2030.5 supports communication in smart grids for energy resources
like ESS, while IEEE 1547 establishes standards for interconnection and interoperability with the grid,
addressing voltage regulation, frequency response, anti-islanding protection and grid support functions.
Successful ESS demonstration projects require seamless integration with other devices highlighting the need
for standardized communication protocols to address interoperability challenges. 

# User Guide

The agents described on this site comprise an interoperability framework to provide integration, control,
and optimization of energy storage systems within a grid environment. This framework is described on
[The Interoperability Framework](/interoperability-framework) page:

Each agent is additionally described on its own page, as follows:

* [Interoperability Agent](/interoperability)
* [Real-time Control](/rt-control)
* [Scheduler](/scheduler)

Links to code and installation instructions can be found at the top of each agent page.
