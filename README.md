# Linux-Namespace-Networking

## Create two Namespaces and connect them using veth

```
# Create two namespaces
sudo ip netns add demo1
sudo ip netns add demo2

# Create a veth pair
sudo ip link add veth-1 type veth peer name veth-2

# Move each end of the veth pair to a different namespace
sudo ip link set veth-1 netns demo1
sudo ip link set veth-2 netns demo2

# Configure IP addresses for the interfaces in each namespace
sudo ip netns exec demo1 ip addr add 192.168.1.1/24 dev veth-1
sudo ip netns exec demo2 ip addr add 192.168.1.2/24 dev veth-2

# Bring up the interfaces
sudo ip netns exec demo1 ip link set dev veth-1 up
sudo ip netns exec demo2 ip link set dev veth-2 up

# two namespaces (demo1 and demo2) are now created and connected using veth pairs (veth-1 and veth-2).

# Test connectivity from demo1 to demo2
sudo ip netns exec demo1 ping 192.168.1.2

# Test connectivity from demo2 to demo1
sudo ip netns exec demo2 ping 192.168.1.1

```

## Create two Namespaces and connect them using Linux bridge ubuntu vm
```
# Create two namespaces
sudo ip netns add demo1
sudo ip netns add demo2

# Create a veth pair
sudo ip link add veth-1 type veth peer name veth-2

# Move each end of the veth pair to a different namespace
sudo ip link set veth-1 netns demo1
sudo ip link set veth-2 netns demo2

# Create a linux bridge
sudo ip link add name brdemo type bridge

# Connect one end of veth pair to the bridge
sudo ip link set veth0 master brdemo


# Configure IP addresses for the interfaces in each namespace
sudo ip netns exec demo1 ip addr add 192.168.1.1/24 dev veth-1
sudo ip netns exec demo2 ip addr add 192.168.1.2/24 dev veth-2

# Bring up the interfaces
sudo ip netns exec demo1 ip link set dev veth-1 up
sudo ip netns exec demo2 ip link set dev veth-2 up

# Bring up the bridge interface
sudo ip link set dev brdemo up


# two namespaces (demo1 and demo2) are now created and connected using veth pairs (veth-1 and veth-2).

# Test connectivity from demo1 to demo2
sudo ip netns exec demo1 ping 192.168.1.2

# Test connectivity from demo2 to demo1
sudo ip netns exec demo2 ping 192.168.1.1

```

