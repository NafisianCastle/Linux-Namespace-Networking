sudo apt install iproute2 net-tools iputils-ping iptables

# create namespace

sudo ip netns add red

sudo ip netns add green

# check created namespace

ip netns

# create bride, set ip and state up

sudo ip link add br0 type bridge

sudo ip addr add 192.168.0.1/16 dev br0

sudo ip link set br0 up

ip link list

# create virtual ethernet(veth)

sudo ip link add veth0 type veth peer name veth-red

sudo ip link add veth1 type veth peer name veth-green

# associated veth ends to different namespace and bridge

sudo ip link set veth0 master br0

sudo ip link set veth1 master br0

sudo ip link set veth-red netns red

sudo ip link set veth-green netns green


# Change veth ends state

sudo ip netns exec red ip link set veth-red up

sudo ip netns exec green ip link set veth-green up

ip link list

sudo ip link set veth0 up

sudo ip link set veth1 up

ip link list

# Change lo of both namespaces' state
sudo ip netns exec red ip link set lo up

sudo ip netns exec green ip link set lo up

sudo ip netns exec green ip link list

# assign ip address to veth in red namespace

sudo ip netns exec red ip addr add 192.168.0.2/16 dev veth-red

# ping 192.168.0.2 from host and from within red
ping -c 2 192.168.0.2 //succesfull 

sudo ip netns exec red ping -c 2 192.168.0.2 //succesfull

# ping to host ip from red namespace
sudo ip netns exec red ip route add default via 192.168.0.1

sudo ip netns exec red bash

route -n // check route table

ip addr show eth0 //from another terminal

ping -c 2 10.42.1.146 // successful

# ping google public DNS from red namespace
//change iptables rule sudo iptables -t nat -A POSTROUTING -s 192.168.0.0/16 -j MASQUERADE
sudo ip netns exec red bash

ping -c 2 8.8.8.8 //successful

# do the same for green namespace
sudo ip netns exec green bash

ip link list

ip link set lo up

ip addr show veth-green

ip addr add 192.168.0.3/16 dev veth-green

# ping veth-green within green namespace, to bridge and host
ping -c 2 192.168.0.3 // successful

ping -c 2 192.168.0.1 // successful

ping -c 2 10.42.1.146 // successful
route

# add a route for green namespace too
ip route add default via 192.168.0.1 route

ping -c 2 10.42.1.146 // successful

# ping to red namespace veth end and google DNS
ping -c 2 192.168.0.2 // successful

ip addr show

ping -c 2 8.8.8.8

