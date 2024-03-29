# Virtual Port Channel (vPC)

After covering Port-Channel let’s know talk about virtual Port Channel (vPC). virtual Port Channel belongs to Multichassis EtherChannel (MCEC) family of technology. vPC allows two NX-OS switches to provide a single port-channel to the downstream devices (Similar to VSS and Stackwise in Catalyst environment). Without vPC, STP blocks one of the ports. In addition, you don’t have high availability.

Note that the downstream switch has no idea that it is connected to two separate switches. With vPC each switch has it’s own control plane and management plane. (Notably, you need to configure vPC switches independently, because management planes are separate). For your note, both VSS and Stackwise create a single logical switch wherein there is a single control plane and a single management plane.

The two peers use a unique **vPC domain** – which does not exists anywhere else – to form vPC. vPC peers track reachability over a layer-3 path called **vPC Peer Keepalive** which is a ping with the payload of UDP-3200. On the other hand, **vPC peer link** using Cisco Fabric Services (CFS) protocol synchronizes the MAC addresses of member ports. Another note about vPC peer link is that this link normally does not play a role in forwarding path. To resume, the downstream devices, connect to both switches using **vPC member ports**. Each member port-channel has a unique vPC number. Last but not least, if a device connects to a vPC peer without vPC configuration, that port on the switch is an **orphaned port** (or a **Non-vPC port)**.

Picture below depicts all the jargons above:

<figure>
  <img src="https://github.com/hosseinoliabak/cisco/assets/31813625/02d2c337-ad21-4de6-ac9d-4ce209e22914" alt="vPC Components" /> <br />
  <figcaption>Figure 1: vPC Components</figcaption>
</figure>

### Enhanced virtual Port Channel

After vPC introduced in NX-OS 4.1(3)N, a N2K FEX could connect to two upstream switches using enhanced vPC. A downstream server can connect to two FEXes using Port-Channel without knowing that it is connected to two different platforms.

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/235332073-9667a869-6789-418b-b322-3f6a2f654181.png" alt="Enhanced vPC">
  <figcaption>Figure 2: Enhanced vPC</figcaption>
</figure>

### vPC with Fabric Peering

In Clos Spine-Leaf design with VXLAN, we don’t connect the leafs together over dedicated peer-links. Instead, we can leverage the current infrastructure for vPC peer-links. Note that Peer keepalive remains out-of-band over mgmt0 or in-band using a dedicated loopback over the L3 fabric.

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/235332190-458a1f6a-a4bf-45e9-ac3b-9342876caead.png" alt="vPC with Fabric Peering">
  <figcaption>Figure 3: vPC with Fabric Peering</figcaption>
</figure>