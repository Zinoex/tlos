# Traffic Light Open Specification - TLOS

Version 0.0.1

__Why another traffic protocol?__

Traffic lights are crucial to our infrastructure and impact the lives of millions if not billions daily, the environment, and the large-scale economies.
Therefore, as a society, we need their optimal performance every second of every day. 
Yet, this is unachievable with many existing protocols due to vendor lock-ins; even for open protocols.

Vendors have to implement the open protocols as required and defined by governments, municipalities, and authorities, but as their incentives are to enforce a vendor lock-in, it is a no-brainer to exploit any loophole in a specification.

To combat all forms of vendor lock-in, we employ the best of the internet: open collaboration as is known from many open-source communities. 
ANY stakeholder including authorities, vendors, universities, traffic engineers, and computer scientists may suggest a change to the protocol free of charge.
Similarly, such a requested change should be open for scrutiny from all stakeholders before being accepted.

To avoid a vendor lock-in, the following principles must be obeyed in the design of the protocol:
1. Modules should have exactly one responsibility
    - The principle break monoliths, allow more vendors, and greater flexibility including improved collaboration across vendors and parallel development.
2. Open for extensions, closed for modification
    - Because we cannot anticipate all technologies of the future, the protocol should be prepared to extend with additional messages.
    But to secure backward compatibility, the interface should be not change existing parts.
3. Small interfaces are preferable
    - The bigger interfaces become, the harder it is maintain or replace (and often violates the first principle). E.g. a traffic radar should not need to know if is communicating with a monitoring or control system.
4. The protocol must target an interface, not an implementation
    - Memory addresses, offsets, etc. are not allowed as these are implementation dependent.

A curious reader may have noticed that these principles are similar to the [SOLID](https://en.wikipedia.org/wiki/SOLID) design principles of object oriented programming.
I, Frederik Mathiesen - Zinoex, am indeed as computer scientist by training, but have plenty of experience working with traffic lights and their protocols. My core vision for the project is to incorporate more computer science principles into the design of traffic light protocols; even if just inspiration for more prevalent protocols. 


## Scope
The protocol is first and foremost a data definition standard amongst modules in a predefined architecture of traffic light systems (see below).
It is not a standard for the hardware and encodings.
A vendor may choose any software and/or hardware implementation of the modules as long as they expose the appropriate interfaces and do their respective _single_ task.

The serialization of the data into a binary, transmitable format will be a standard, open, compact, and lightweight protocol.
The scope of this project is not to reduce transmission sizes; it can be done by selecting the appropriate framework, using its structures as intended (e.g. bitmasks sent as numbers and arrays encoded as strings are not allowed), and applying compression.

A concern raised is the size of binary data will be too large for transmission or that the compression/decompression will be too computationally expensive.
This is a malplaced concern as even implementations of efficient, binary serialization formats like [msgpack](https://github.com/hideakitai/MsgPack) or [protobuf](https://github.com/nanopb/nanopb) exist for embedded systems, and a compression algorithm like [lz4](https://www.embedded.com/speeding-over-the-air-latency-for-iot-applications-with-compression/) is fast, has a good compression ratio, and produces small binaries, making it perfect for IoT.


## Architecture
<embed src="architecture.pdf" type="application/pdf">
    <p>This viewer does not support PDFs.</p>
</embed>

### Signal
A single traffic signal, e.g. a 3-light signal, and exposes an extremely simple interface.
The interface includes:
- Requesting the type of signal including control modes
- Setting the control mode, i.e. red, red amber, amber, green, bliking yellow, etc. for a standard 3-signal
- Reading the health of the signal (LEDs, bulb, etc.)

### Controller
The task of the controller is to receive commands from the scheduler, validate its safety, and forward the command to the signals based on a signal group configuration; it is _NOT_ allowed to compute the schedule itself.
Each controller only handles exactly one intersection, even for coordinated traffic lights.
Additionally, the controller observes the response from the signals to check their ability to execute the command.

Safety checks:
- Conflicting movements
- Intra- and inter-signal timing
- Individual signal health

### Coordinator
The coordinator encompasses a single or multiple coordinated intersections and is the on-location entrypoint for the traffic center software, i.e. supervisors.

The coordinator consists of 3 major components:
- Management: Managing the topology definition, sensor configuration, etc. This is controllable both from the traffic central and a physical interface.
- Monitoring: Monitor the health of the components, analyze the sensor data for computing alarms, and perform security surveillance (SIEM).
- Scheduler: Plan the light schedule for one or more intersections. Can be time schedules, extension-based schedules (by data from the monitoring), or computational tool-based schedules. 

### Sensor
A sensor is defined as on-location sensory equipment for reading the traffic situation.
- V2I
- Traffic radar
- Cameras
- Pedestrian buttons
- Detector loops

Note that for sensor equipment not supporting this protocol, it is recommended to develop a stand-alone adaptor rather than letting the coordinator support its direct communication.
The reason is that a coordinator not implementing proprietary or specific protocols are more prone to a vendor lock-in.


### Supervisor
A supervisor is a very generic piece of software. It can be a traffic center system, a public data API, or an operator supervision.
The core idea is that they can monitor and control the intersection.
Because of the versatility, the coordinator must be designed for multiple concurrent supervisors.

The interface between the supervisor and the coordinator is split into three topics:
- Control: The only topic allowed to change the operation of the intersection. This interface should be strictly controlled with authorization (see Cross-cutting concerns -> Security).
- Health: Useful for operators that maintain but not own the intersection.
- Data: A forwarding of the sensory data through the coordinator. Useful for traffic center systems or public data APIs.

The reason for splitting the topics is their applicability towards different types of supervisors where the access to each topic should be role dependent.


## Cross-cutting concerns
- Communication method - WebSocket
    - No requirement for the underlying method (ethernet, WiFi, fiber, broadband cellular network)
    - Ping / pong frames - heartbeats
    - Speeds are really not an issue, but latency is, and there is little overhead and low latencies for websockets
    - Encryption
- Authentication
- Authorization
- Failure reporting


## Versioning
The project uses [Semantic Versioning 2.0.0](https://semver.org/).


## Disclaimer
This project is not responsible for any consequences of the specification to any third party.

