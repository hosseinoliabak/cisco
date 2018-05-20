# Network Models
What is a Model? 
* We use models to represent how networks function 
* There are 2 very popular network models: The OSI 7-Layer Model and the TCP/IP Model

## OSI Model vs. TCP/IP Model 

* OSI Model: 1-Physical; 2- Data Link; 3- Network; 4- Transport; 5- Session; 6- Presentation; 7- Application 
* TCP/IP Model: 1- Network Interface; 2- Internet; 3- Transport 4- Application

### OSI Model

| Layer | Protocol Data Unit | Characteristics | Standards bodies | 
| --- | --- | --- | --- | 
| 7. Application | Data (APDU) | Closest to users | IETF/RFC |  
| 6. Presentation | Data (PPDU) | Data format, compression and encryption | IETF/RFC | 
| 5. Session | Data (SPDU) | Keep an open link between apps. | IETF/RFC | 
| 4. Transport | Segment, Datagram | Virtual circuits, reliability and ports | IETF/RFC | 
| 3. Network | Packet | Logical addressing and path determination | IETF/RFC | 
| 2. Data Link | Frame | Physical addressing, error detection | IEEE 802 |
| 1. Physical | Bit | Transporting the bits | IEEE 802 |

* Protocol
* Service

### TCP/IP Model


## 1. Physical Layer

### Cabling and Topology 

#### Twisted-Pair Cabling 
* Unshielded twisted-pair (UTP) 
* Shielded twisted-pair (STP) 
* Straight-Through Cables: CAT 5 UTP cabling usually uses only four wires when sending and receiving information on the
network. The four wires of the eight that are used are wires 1, 2, 3, and 6. 

    ![straight](https://user-images.githubusercontent.com/31813625/40263577-85c09174-5ae2-11e8-94e0-a822b1392b2c.png)

* Crossover Cables 

    ![cross](https://user-images.githubusercontent.com/31813625/40263578-85e20bc4-5ae2-11e8-86c2-874ccb2f2654.png)

* TIA 568A and TIA 568B Standards Although only four of the wires are used to send and receive data in most environments
today, some of the newer standards use all eight wires. Therefore, it is important to know the order of all eight wires
in a UTP cable. There are two popular wiring standards today, 568A and 568B, but 568B is the more popular

#### Coaxial Cabling

* Better shielding than twisted-pair
* Lower attenuation & cross talk
* Can carry 1~2 Gbps in 1 km
* Telcos used it for their backbone communication. It is replaced by fiber optics
* Cable TV companies use it for TV and data delivery services

Today, your focus should migrate from the 50ohm coax of early Ethernet to the 75ohm coax of early
(and modern, of course) cable television. The reason for this is that while coax in the Ethernet world is all but a
thing of the past, RG-6 or CATV coax is alive and well in the world of broadband cable (cable modem) technology.
The connectors used with coax in this environment are the same F-Type connectors used for standard cable television
connectivity. In fact, the data rides on the same medium, just over different frequencies There are two types of coax
cabling (Both are 10 Mbps):
  * Thinnet (RG-58); ¼ inch thick; is used for short-distance communication; Thinnet connects directly to a
  workstation’s network adapter card using a British naval connector (BNC) and uses the network adapter card’s internal
  transceiver. The maximum length of thinnet is 185 meters. The BNC-T connector is used to connect to coax cable from
  either side (so that the cable length can continue on), while a third end of the connector tees out to have a cable
  length connect to the network card on the client machine
  
    ![thinnet](https://user-images.githubusercontent.com/31813625/40263406-c93ff17c-5adf-11e8-86a3-b2fb28334eca.png)
    
  * Thicknet (RG-8); ½ inch thick; maximum cable length of 500 meters and usually is used as a backbone to connect
  several smaller thinnet-based networks; A transceiver often is connected directly to the thicknet cable using a
  connector known as a vampire tap. Connection from the transceiver to the network adapter card is made using a drop
  cable to connect to the adapter unit interface (AUI) port connector
  
    ![thicknet](https://user-images.githubusercontent.com/31813625/40263403-c0326b82-5adf-11e8-8a11-27f477af5224.png)    

#### Fiber Optic Cabling

An optical fiber consists of an extremely thin cylinder of glass, called the core, surrounded by a concentric layer of
glass, known as the cladding. There are two fibers per cable—one to transmit and one to receive 
* Single-mode fiber (SMF) Uses a single ray of light (laser), known as a mode, to carry the transmission over long
distances (yellow) 
* Multimode fiber (MMF) Uses multiple rays of light (modes) simultaneously, with each ray of light running at a different
reflection angle to carry the transmission over short distances (orange; green) 

Fiber-optic cables can use many types of connectors, but the Network+ exam is concerned only with the 4 major
connector types: 
* Straight-tip connector (ST): based on the BNC-style connector but has a fiber-optic cable instead of a copper cable 
* Subscriber connector (SC): square and somewhat similar to an RJ-45 connector 
* Lucent Connector (LC): Small connectors; simple and stable; blue: single mode; Beige: Multi mode; becoming the most popular one 
* MT-Rj; Fiber to the deck connector (FTTD); having 2 fibers and being just as small as a Rj45 connector; not so popular anymore  

Fiber optic polishing 
  * PC 
  * UPC 
  * APC

#### Power Lines
* Power Lines have been used by Electric companies for data transmission for many years (Low rate data communication
for control and dispatching)
* Power Lines can be used for broadband data communication
  * Providing broadband access to homes using Electric grid as the access network
  * Creating a data communication network inside homes

#### Wireless transmission
* Main Drivers
  * Communication without a physical attachment such as wires or fibers
  * Mobility of users
  * Ease of deployment
  * Reduce cost of communication (For example, providing connectivity services to remote villages)
* Higher frequencies (shorter wave lengths) have much larger available bandwidth
  * Radio range of frequencies (10KHz to 10MHz) usually used for broadcast communication
  * Microwave range of frequencies (10MHz to 10GHz) usually used for point to point long distance transmission and local two-way communications
  * Most radio frequency bands are regulated. Using frequency spectrum for commercial purposes is usually subject to a regulatory fee
    * Since 1994, US government has earned more than 60 billion US$ from spectrum licensing fees
    * Irancell has paid 300 million Euro for its spectrum license to Communications Regulatory Agency (CRA)
  * There are license free bands available in most countries (ISM bands)   


#### Wireless Optical Transmission
Idea: Light as the information carrier for free space communication
* Indoor applications: Wireless LAN, IrDA standard
* Outdoor application: Building to building communication

#### Satellite Communication
* Satellites can be used as a wireless node in the sky that can receive, amplify, process and transmit communication signals
* They are mainly used in three orbit ranges and therefore have different rotation period around earth
* Satellites generally have multiple transponders each with a given bandwidth/capacity for data transport
* They usually use "spot-beams" to focus their transmission on a given region
* Most communication satellites are at Geostationary orbit
* Satellite operational frequency ranges:

  | Band | Downlink | Uplink | Bandwidth | Problems |
  | --- | --- | --- | --- | --- |
  | L | 1.5 GHz | 1.6 GHz | 15 MHz | Low BW, Crowded |
  | S | 1.9 GHz | 2.2 GHz | 70 MHz | Low BW, Crowded |
  | C | 4.0 GHz | 6.0 GHz | 500 MHz | Terrestrial Interference |
  | Ku | 11 GHz | 14GHz | 500 MHz | Rain |
  | Ka | 20 GHz | 30 GHz | 3500MHz | Rain, Equipment cost |


### Introduction to Structured Cabling 

* The Electronic Industries Association and the newer Telecommunications Industry Alliance (EIA/TIA) - nowadays only
TIA - is the standards body that creates the Physical layer specifications for Ethernet 
* Telecommunication Closet/Equipment Room 
  * Patch panel: is one end of horizontal run 
    * 110 punchdowns 
  * Patch cable (equipment cord) 
    * RJ-45 crimps 
* Horizontal Run: 
  * fixed 
  * Maximum 90 meters 
* Work Area 
  * wall outlet 
    * 110 punchdowns 
  * Work area patch cable 
    * RJ-45 crimps 

#### MDF, IDF, Dmarc, and the Equipment Room
* **Rack Unit (RU or U):** 1.75 in (4.44 cm) 
* **Main Distribution Frame (MDF):** a cable rack that interconnects and manages the telecommunications wiring between itself and any number of IDFs 
* **Intermediate Distribution Frame (IDF):** a free-standing or wall-mounted rack for managing and interconnecting the telecommunications cable between end user devices and a MDF 
* **Demarc:** Also called point of demarcation (POD), or demarc extension; it is the physical point at which the public network of a telecommunications company (i.e., a phone or cable company) ends and the private network of a customer begins - this is usually where the cable physically enters a building 
   
#### Testing Cable 
* Microscanner 
  * Wire map 
  * Continuity 
  * Distance: Time Domain Reflectometer (TDR) 
    * OTDR for Fiber Optics 
  * Crosstalk: electromagnetic interference (EMI) 
  * Attenuation: As a signal moves through any medium, the medium itself will degrade the signal—a phenomenon known as attenuation that’s common in all kinds of networks 
  * Modal distortion: multimode fiber’s issue 

#### Using a Toner and Probe 
How to find cables without labels? 
* Fox and Hound (Tone Generator and Tone Probe): 
  * Tone generators: create the signal 
  * Tone probes: translate the signal into an audible tone 
  * RJ-11 snaps into RJ-45 just fine

#### Baseband and Broadband 
* Baseband: Sends digital signals through the media as a single channel that uses the entire bandwidth of the media.
The signal is delivered as a pulse of electricity or light, depending on the type of cabling being used.
Baseband communication is also bidirectional, which means that the same channel can be used to send and receive signals 
* Broadband: Sends information in the form of an analog signal, which flows as electromagnetic waves or optical waves.
Each transmission is assigned to a portion of the bandwidth, so unlike with baseband communication, it is possible to
have multiple transmissions at the same time, with each transmission being assigned its own channel or frequency.
Broadband communication is unidirectional, so in order to send and receive, two pathways will need to be used.
This can be accomplished either by assigning a frequency for sending and assigning a frequency for receiving along the
same cable or by using two cables, one for sending and one for receiving 

## 2. Data Link Layer
### Purpose
* Ensures reliable, efficient communication between neighbor nodes
* Four specific functions:
  * Provide services to network layer
  * Framing: Since the physical layer merely accepts and transmits a stream of bits
without any regard to meaning or structure, it is upto the data link layer
to create and recognize frame boundaries. This can be accomplished by attaching
special bit patterns to the beginning and end of the frame. If these bit
patterns can accidentally occur in data, special care must be taken to make
sure these patterns are not incorrectly interpreted as frame delimiters.
  * Error handling (Error Detection and Correction)
    * The most common error control method is to compute and append some form of a
checksum to each outgoing frame at the sender's data link layer and to recompute
the checksum and verify it with the received checksum at the receiver's side.
If both of them match, then the frame is correctly received; else it is erroneous.
  * Flow control
    * Consider a situation in which the sender transmits frames faster than the
receiver can accept them. If the sender keeps pumping out frames at high rate,
at some point the receiver will be completely swamped and will start losing
some frames. This problem may be solved by introducing flow control. Most flow
control protocols contain a feedback mechanism to inform the sender when it should transmit the next frame.



### Ethernet Basics 


#### What is Ethernet (IEEE 802.3)? 
* 10Base2: 10Mbps; baseband; 200m cable length (thinnet cable); 30 hosts per segment 
* 10Base5: 10Mbps; baseband; 200m cable length (thicknet cable); 100 hosts per segment 
* 10BaseT architecture: 10Mbps; baseband; Twisted-pair cable (CAT 3 cable or better) 

#### Early Ethernet 
##### Token Ring 
A big competitor to Ethernet in the past was Token Ring (invented by IBM), which runs at 4 Mbps or 16 Mbps.
Token Ring is a network architecture that uses a star ring topology; Token Ring uses the token-passing access method 

A Token Ring MAU is a hub-type device for Token Ring networks with some features that make it a little bit different from a hub—for example, a MAU regenerates the signal when it reaches the MAU 

#### 10BaseT 

10Mbps; baseband; Twisted-pair cable (CAT 3 cable or better); Maximum of 1024 nodes per switch 

Page Break
 

#### Modern Ethernet 
##### Modern Ethernet, and Duplex 
* Fast Ethernet (100BaseX) (IEEE 802.3u) 
  * 100BaseTX: four wires in the CAT 5 cabling; 1024 node per hub; 100m 
  * 100BaseFX: two strands of fiber; 1024 node per hub; 2km 
* Gigabit Ethernet 
  * IEEE 802.3z: fiber optic or coaxial cabling 
    * 1000BaseSX: MMF; 500m
    * 1000BaseLX: SMF; 5km
    * 1000BaseCX: twinax coaxial cable (up to 25 meters) 
  * IEEE 802.3ab (1000BaseTX): >CAT 5e; all 8 wires in twisted-pair cabling; CAT 6: 100m 
* 10-Gigabit Ethernet (IEEE 802.3ae): 
  * Ethernet LAN 
    * 10GBaseR: short-range MMF; <100m 
    * 10GBaseLR: long-range SMF (light wave length: 1310nm); <10km 
    * 10GBaseER: extra-long-range SMF (light wave length: 1550nm); <40km 
    * 10GBaseT: CAT 6 UTP; <100m 
  * SONET WAN 
    * 10GBaseSW: short-range MMF to connect to SONET; W means WAN 
    * 10GBaseLW: long-range SMF to connect to SONET; W means WAN 
    * 10GBaseEW: extra-long-range SMF to connect to SONET; W means WAN 

##### Transmission methods 
* Simplex: Allows communication in one direction only. You will only be able to send or receive with a simplex device—not both directions. It is either one way or the other 
* Half duplex: Allows communication in both directions (send and receive), but not at the same time. A network card set to half duplex will not be able to receive data while sending data. Using the half-duplex setting can slow down communication if your device does support full duplex 
* Full Duplex: Allows communication in both directions at the same time. If a network card supports full duplex, it will be able to receive data when data is being sent because all four pairs of wires are used. If you make sure that a network card that supports full duplex is set to full duplex, you will notice a big difference in throughput if the device is set to half duplex 

#### Switch Backbones 
* High-speed switches connecting secondary switches are backbones 
* SFP vs. GBIC Transceivers: SFP, also called mini-GBIC, was designed after the GBIC interface. It is half the volume of GBIC and can be configured double number of ports on the same panel. Other basic functions is the almost the same with the GBIC 
* Bridge Loop 
* Spanning Tree Protocol automatically shuts down the bridge oops 

#### Layer 2 Ethernet Frame
![2000px-ethernet_802 1q_insert svg](https://user-images.githubusercontent.com/31813625/39190013-b1843ba2-47a1-11e8-8bb1-afdcb5eb6f59.png
)

 Ethernet is a broadcast architecture
• Collision will occur

### Access Methods 

An access method determines how a host will place data on the wire — does the host have to wait its turn or can it just place the data on the wire whenever it wants 

#### Carrier sense multiple access with collision detection (CSMA/CD) 

To summarize, CSMA/CD provides that before a host sends data on the network, it will "sense" (CS) the wire to ensure
that the wire is free of traffic. Multiple systems have equal access to the wire (MA), and if there is a collision,
a host will detect that collision (CD) and retransmit the data

* 802.3 

#### Carrier sense multiple access with collision avoidance (CSMA/CA) 

CSMA/CD deals with transmissions after a collision has occurred. With CSMA/CA, before a host sends data on the PHY,
typically wireless (802.11), it will "sense" the PHY as well to see if the it is free of signals. If the PHY is free,
it will try to "avoid" a collision by sending a piece of "dummy" data first to see whether it collides with any other data.
If it does not collide, the host assumes "If my dummy data did not collide, then the real data will not collide," and it
submits the real data on the PHY 

#### Token Passing 

With both CSMA/CD and CSMA/CA, the possibility of collisions is always there, and the more hosts that are placed on the
wire, the greater the chances of collisions, because you have more systems "waiting"’ for the wire to become free so
that they can send their data. 

With token passing, there is an empty packet running around on the wire—the "token." In order to place data on the wire,
you need to wait for the token; once you have the token and it is free of data, you can place your data on the wire.
Since there is only one token and a host needs to have the token to “talk,” it is impossible to have collisions in a
token-passing environment. For example, if Workstation 1 wants to send data on the wire, the workstation would wait for
the token, which is circling the network millions of times per second. Once the token has reached Workstation 1, the
workstation would take the token off the network, fill it with data, mark the token as being used so that no other
systems try to fill the token with data, and then place the token back on the wire heading for the destination host.
All systems will look at the data, but they will not process it, since it is not destined for them. However, the system
that is the intended destination will read the data and send the token back to the sender as a confirmation. Once the
token has reached the original sender, the token is unflagged as being used and released as an empty token onto the
network

## 3. Network Layer
Role of network layer: routing packets
* Deals with end-to-end communication
* Logical addressing
* Aware of network topology
* Choose appropriate route
* Communicates with transport and data-link layers

### Routed protocols vs routing protocols
* Routed protocol: IP, AppleTalk, and IPX
* Routing Protocols: RIP, EIGRP, OSPF, IS-IS, BGP

## 4. Transport Layer
Tasks:
* Provide reliable and cost-effective end-to-end communication service to the application layer
* Independent of used network (shielding)
* Boundary between network and applications
* TPDU (transport protocol data unit) AKA "segment" denotes message sent from entity to entity
* Can be both connection oriented and connectionless service

### Transport Services
* *Transport Services* vs *Data Link* Services
  * Similarities:
    * Both provide point-to-point connection
      * point-to-point means one path to the destination. When a packet goes in one end, it must come out the other end!
    * Both have to deal with error control, sequencing, flow control, retransmission, etc
  * Differences:
    * Transport connection is indirect
    * Network has memory: packets may be stored and and arrive with varying delays and out of order
      * In Data link, when we send first and send frame to PHY, we receive them in order. But in transport layer, when sending segments
    maybe we receive the second segment prior to th first segment
    * Many connections have to be managed (instead of a fixed number of links)
* Primitives for a simple transport service:
  * LISTEN: block until some process connects
  * CONNECT
  * SEND
  * RECEIVE: block until data TPDU arrives
  * DISCONNECT
* TCP addressing:
  * IP protocol number, TCP/UDP port number
  
### Transmission Control Protocol (TCP)

### User Datagram Protocol

## 5. Session Layer


## 6. Presentation Layer

## 7. Application Layer