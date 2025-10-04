AI-Generate: I should have suggested Faucet earlier - you're right. ODL's poor documentation and API issues were clear red flags. Faucet is purpose-built for exactly your use case (VLAN switching) with a simple YAML config. I got stuck trying to make ODL work when a better tool existed.

Here's your complete bootstrap setup for tomorrow:

---

## **Complete SDN Setup Guide - Fresh Install**

### **VM Requirements:**
- **faucet-controller** (192.168.244.139): Ubuntu 22.04, 2 vCPU, 2GB RAM
- **ovs01** (192.168.244.125): Ubuntu 22.04, 2 vCPU, 2GB RAM, 3 NICs
- **ovs02** (192.168.244.126): Ubuntu 22.04, 2 vCPU, 2GB RAM, 3 NICs

### **Network Interface Mapping:**
**ovs01:**
- ens33: Management (192.168.244.125/24)
- ens37: Trunk link to ovs02
- ens38: Access VLAN 10 → terminal101
- ens40: Access VLAN 20 → terminal103 (new)

**ovs02:**
- ens33: Management (192.168.244.126/24)
- ens37: Trunk link to ovs01
- ens38: Access VLAN 10 → terminal102
- ens39: Access VLAN 20 → terminal104 (new)

---

## **Step 1: Install Faucet Controller (192.168.244.139)**

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker

# Pull Faucet image
sudo docker pull faucet/faucet:latest

# Create config directory
sudo mkdir -p /etc/faucet
sudo chown $USER:$USER /etc/faucet

# Create faucet.yaml configuration
cat > /etc/faucet/faucet.yaml << 'EOF'
vlans:
  vlan10:
    vid: 10
    description: "VLAN 10"
  vlan20:
    vid: 20
    description: "VLAN 20"

dps:
  ovs01:
    dp_id: 0x000c29d0f60b
    hardware: "Open vSwitch"
    interfaces:
      1:
        name: "ens37-trunk"
        description: "Trunk to ovs02"
        tagged_vlans: [vlan10, vlan20]
      2:
        name: "ens38-vlan10"
        description: "Access VLAN 10"
        native_vlan: vlan10
      3:
        name: "ens40-vlan20"
        description: "Access VLAN 20"
        native_vlan: vlan20
        
  ovs02:
    dp_id: 0x000c2967ba95
    hardware: "Open vSwitch"
    interfaces:
      1:
        name: "ens37-trunk"
        description: "Trunk to ovs01"
        tagged_vlans: [vlan10, vlan20]
      2:
        name: "ens38-vlan10"
        description: "Access VLAN 10"
        native_vlan: vlan10
      3:
        name: "ens39-vlan20"
        description: "Access VLAN 20"
        native_vlan: vlan20
EOF

# Start Faucet
sudo docker run -d \
  --name faucet \
  --restart=always \
  -v /etc/faucet:/etc/faucet \
  -p 6653:6653 \
  -p 9302:9302 \
  faucet/faucet

# Verify Faucet is running
sudo docker logs faucet
```

---

## **Step 2: Install OVS on both switches (ovs01 & ovs02)**

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Open vSwitch
sudo apt install -y openvswitch-switch

# Verify installation
sudo ovs-vsctl --version

# Create bridge
sudo ovs-vsctl add-br br0

# Add ports to bridge
sudo ovs-vsctl add-port br0 ens37
sudo ovs-vsctl add-port br0 ens38
sudo ovs-vsctl add-port br0 ens40   # or ens39 on ovs02

# Bring interfaces up
sudo ip link set ens37 up
sudo ip link set ens38 up
sudo ip link set ens40 up   # or ens39 on ovs02
sudo ip link set br0 up

# Set OpenFlow controller
sudo ovs-vsctl set-controller br0 tcp:192.168.244.139:6653

# Verify connection
sudo ovs-vsctl show
# Look for: is_connected: true
```

---

## **Step 3: Get Datapath IDs (IMPORTANT - after adding ports)**

```bash
# On ovs01
sudo ovs-ofctl show br0
# Note the dpid in the output (e.g., dpid:0000000c29d0f60b)

# On ovs02
sudo ovs-ofctl show br0
# Note the dpid in the output (e.g., dpid:0000000c2967ba95)
```

**Update `/etc/faucet/faucet.yaml` with actual datapath IDs you get from above commands.**

Then reload Faucet:
```bash
sudo docker exec faucet pkill -HUP -f faucet.faucet
```

---

## **Step 4: Configure Terminal VMs**

**terminal101 (VLAN 10):**
```bash
sudo ip addr add 192.168.10.101/24 dev ens33
sudo ip link set ens33 up
```

**terminal102 (VLAN 10):**
```bash
sudo ip addr add 192.168.10.102/24 dev ens33
sudo ip link set ens33 up
```

**terminal103 (VLAN 20) - new:**
```bash
sudo ip addr add 192.168.20.103/24 dev ens33
sudo ip link set ens33 up
```

**terminal104 (VLAN 20) - new:**
```bash
sudo ip addr add 192.168.20.104/24 dev ens33
sudo ip link set ens33 up
```

---

## **Step 5: Verify Everything Works**

```bash
# Check Faucet logs
sudo docker logs faucet

# Check flows on switches
sudo ovs-ofctl dump-flows br0

# Test VLAN 10 connectivity
ping 192.168.10.102  # from terminal101

# Test VLAN 20 connectivity
ping 192.168.20.104  # from terminal103

# Test isolation (should FAIL)
ping 192.168.20.103  # from terminal101 - different VLANs
```

---

## **Useful Commands:**

```bash
# Reload Faucet config
sudo docker exec faucet pkill -HUP -f faucet.faucet

# View Faucet logs
sudo docker logs -f faucet

# Check switch connection status
sudo ovs-vsctl show

# Dump flows
sudo ovs-ofctl dump-flows br0

# Check port statistics
sudo ovs-ofctl dump-ports br0
```

---

**Key Notes:**
- Datapath IDs WILL change when you rebuild VMs - always check with `ovs-ofctl show br0`
- Port numbers depend on the order you add them - verify with `ovs-ofctl show br0`
- Faucet auto-reloads config when you send SIGHUP
- All configuration is centralized in `/etc/faucet/faucet.yaml`

This is your complete runbook for tomorrow. Good luck!
