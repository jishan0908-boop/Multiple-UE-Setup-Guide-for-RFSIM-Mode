# Multiple UE Setup Guide for RFSIM Mode

## To run UE2 with an existing UE1 in RFsim mode the Cmd I used.

### Create Network Namespace and Virtual Ethernet Pair for UE2
```
# Clean up if exists
sudo ip netns delete ueNameSpace2 2>/dev/null
sudo ip link delete v-eth2 2>/dev/null

# Create namespace
sudo ip netns add ueNameSpace2

# Create virtual ethernet pair (v-eth2 <-> v-ue2)
sudo ip link add v-eth2 type veth peer name v-ue2

# Move v-ue2 into the namespace
sudo ip link set v-ue2 netns ueNameSpace2

# Configure host side (v-eth2)
sudo ip addr add 10.201.1.1/24 dev v-eth2
sudo ip link set v-eth2 up

# Configure NAT and forwarding (use eno1 - your interface)
sudo iptables -t nat -A POSTROUTING -s 10.201.1.0/255.255.255.0 -o eno1 -j MASQUERADE
sudo iptables -A FORWARD -i eno1 -o v-eth2 -j ACCEPT
sudo iptables -A FORWARD -o eno1 -i v-eth2 -j ACCEPT

# Configure namespace side (v-ue2)
sudo ip netns exec ueNameSpace2 ip link set dev lo up
sudo ip netns exec ueNameSpace2 ip addr add 10.201.1.2/24 dev v-ue2
sudo ip netns exec ueNameSpace2 ip link set v-ue2 up
```
### Verify Setup
```
# Check namespace exists
sudo ip netns list

# Check interfaces
ip link show | grep v-eth2
sudo ip netns exec ueNameSpace2 ip link show

# Test connectivity between host and namespace
ping -c 3 10.201.1.2
sudo ip netns exec ueNameSpace2 ping -c 3 10.201.1.1
```

### Run the 2nd UE
```
sudo ip netns exec ueNameSpace2 ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --uicc0.imsi 001010000000002 --rfsim --rfsimulator.serveraddr 10.201.1.1
```
### After Connected to test it is working or not
```
sudo ip netns exec ueNameSpace2 ip addr show
sudo ip netns exec ueNameSpace2 ping 8.8.8.8
sudo ip netns exec ueNameSpace2 curl http://www.google.com
```
