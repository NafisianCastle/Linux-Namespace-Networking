# Prerequisites
sudo apt update

sudo apt upgrade -y

sudo apt install iproute2 net-tools iputils-ping iptables
# Create a Linux bridge:
sudo ip link add dev v-net type bridge

sudo ip link set dev v-net up

# Assign an IP address to the bridge interface 'v-net':
sudo ip address add 10.0.0.1/24 dev v-net

# create namespace

sudo ip netns add red-ns

sudo ip netns add green-ns

### check created namespace

sudo ip netns list

# Create virtual Ethernet pairs:
sudo ip link add veth-green-ns type veth peer name veth-green-br

sudo ip link add veth-red-ns type veth peer name veth-red-br

# Move each end of veth cable to a different namespace:
sudo ip link set dev veth-green-ns netns green-ns

sudo ip link set dev veth-red-ns netns red-ns

# Add the other end of the virtual interfaces to the bridge:
sudo ip link set dev veth-green-br master v-net

sudo ip link set dev veth-red-br master v-net

# Set the bridge interfaces up:
sudo ip link set dev veth-green-br up

sudo ip link set dev veth-red-br up

# Set the namespace interfaces up:
sudo ip netns exec green-ns ip link set dev veth-green-ns up
sudo ip netns exec red-ns ip link set dev veth-red-ns up

# Assign IP addresses to the virtual interfaces within each namespace and set the default routes:
sudo ip netns exec green-ns ip address add 10.0.0.11/24 dev veth-green-ns
sudo ip netns exec green-ns ip route add default via 10.0.0.1

sudo ip netns exec red-ns ip address add 10.0.0.21/24 dev veth-red-ns
sudo ip netns exec red-ns ip route add default via 10.0.0.1

#  Firewall rules:
sudo iptables --append FORWARD --in-interface v-net --jump ACCEPT

sudo iptables --append FORWARD --out-interface v-net --jump ACCEPT

# Test Connectivity
sudo ip netns exec red-ns ping -c 2 10.0.0.11
sudo ip netns exec green-ns ping -c 2 10.0.0.21


# ping google public DNS from red namespace

### change iptables rule 

sudo iptables -t nat -A POSTROUTING -s 192.168.0.0/16 -j MASQUERADE

sudo ip netns exec red-ns bash

ping -c 2 8.8.8.8 //successful

sudo ip netns exec green-ns bash

ping -c 2 8.8.8.8 //successful

