# NAT and Port Forwarding

Internal VMs need internet access. External clients need to reach internal services. Use iptables MASQUERADE for outbound, DNAT for inbound.

## When to Use
- A host in an internal network (e.g. `10.10.0.0/24`) needs to reach the internet.
- An external client needs to reach a service running inside the internal network.

## How
### Outbound NAT (VMs → Internet)
```bash
iptables -t nat -A POSTROUTING -s 10.10.0.0/24 -o ombr0 -j MASQUERADE
```
- `POSTROUTING` — modify packet as it leaves
- `-s 10.10.0.0/24` — source is internal network
- `-o ombr0` — outgoing interface (public)
- `MASQUERADE` — replace source IP with interface IP

### Inbound Port Forwarding (Internet → VM)
```bash
iptables -t nat -A PREROUTING -i ombr0 -p tcp --dport 8080 -j DNAT --to-destination 10.10.0.5:80
```
- `PREROUTING` — modify packet before routing decision
- `-i ombr0` — incoming interface (public)
- `--dport 8080` — external port
- `DNAT --to-destination` — rewrite destination to internal VM

### Persistence
```bash
apt install iptables-persistent
netfilter-persistent save
```

## Rule
NAT is stateful. Return traffic works automatically because the kernel tracks connections. No need for reverse rules.
