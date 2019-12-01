# 2-way-transparency
How to use a single public IP on multiple devices

Note: See Diagram for reference

Note 2: This assumes that Pi 1 is running 2 services - one on port 8081, and one on port 8082. If anyone tries to hit 1.1.1.2 from the internet on those 2 ports, they'll be talking to Pi 1. If they hit any other port, they'll be talking to the customer router

Generic Steps:

* Configure Static IPs on both Pis
* Enable routing on both Pis (echo 1 > /proc/sys/net/ipv4/ip_forward)

Pi 1:

	ip route add default via 1.1.1.1

	iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
	iptables -t nat -A POSTROUTING -o eth1 -s 1.1.1.1 -j MASQUERADE

	# All UDP Ports
	iptables -t nat -A PREROUTING -i eth0 -p udp -j DNAT --to-destination 192.168.90.6

	# Some TCP Ports
	iptables -t nat -A PREROUTING -i eth0 -p tcp -m multiport ! --dport 8081,8082 -j DNAT --to-destination 192.168.90.6

Pi 2:

	ip route add default via 192.168.90.5

	iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
	iptables -t nat -A POSTROUTING -o eth1 -s 192.168.90.5 -j MASQUERADE

	# Forwards EVERYTHING to Customer Router
	iptables -t nat -A PREROUTING -i eth0 -j DNAT --to-destination 192.168.90.6
