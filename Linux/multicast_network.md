Ctrl+Shift V

[Reference](https://low-orbit.net/linux-how-to-join-multicast-group)

## ip Ranges and Groups

| Ranges:   |                 |                                          |
| --------- | --------------- | ---------------------------------------- |
| 224.0.0.0 | 239.255.255.255 | Class D addresses are used for multicast |
| 224.0.0.0 | 224.0.0.255     | Reserved for local use, never forwarded  |
| 239.0.0.0 | 239.255.255.255 | Reserved for administrative scoping      |

| Groups:    |                             |
| ---------- | --------------------------- |
| 224.0.0.1  | all hosts group             |
| 224.0.0.2  | all multicast routers group |
| 239.0.0.4  | all DVMRP routers           |
| 239.0.0.5  | all OSPF routers            |
| 239.0.0.13 | all PIM routers             |

To find all hosts with multicast enabled, run:
```console
ping 224.0.0.1
```

## Setting Up Multicast on Linux

Enable multicast on the interface like this:

```console
sudo ifconfig enp3s0 multicast
```

Add a route for a class D network:
```console
sudo smcroute -j enp3s0 239.168.1.50
// sudo route add -net 224.0.0.0/8 dev enp3s0
```

Sniff multicast traffic on this interface. If you see any packets it is working:
```console
sudo tcpdump -i enp3s0 ip multicast
```
The following will show multicast addresses:
```console
ip maddress show
ip maddress show | grep 224.2.2.4
```
