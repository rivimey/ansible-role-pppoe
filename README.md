# PPPoE Service Role

Create a service to manage a PPPoE connection on the targeted system.

## Requirements

The role depends on the systemd, pppd, and pppoe packages, but otherwise has no dependencies.

## Role Variables

Available variables are listed below:

Ethernet device used to talk to pppoe endpoint:

```yaml
pppoe_eth_dev: eth7
```

Enable use of ethernet link MTUs greater than 1500, that is, using "jumbo frames". Because some
network configurations don't support jumbo frames, this may not work for you.

```yaml
pppoe_boost_mtu: true
```

The MTU configured on the pppoe_eth_dev interface. If this interface is being used for other 
purposes, doing this could interfere -- if so, set this to 0 to omit any change to the link
MTU.

```yaml
pppoe_eth_mtu: 1508
```

The MTU configured on the created pppoe link. Ideally this would match the MTU used on the
main network, to avoid fragmentation, but may be constrained by the configuration of the PPPoE
peer and remote network.

On pure Ethernet, the official Ethernet max packet size of 1500 bytes allows for 1460 bytes of
IP payload (20 bytes IP header, 20 bytes TCP header). Putting that in a PPPoE connection adds 8
bytes of ppp header, so the max MTU must be lowered to 1492 bytes. Adding the Ethernet header
to a 1492 byte pppoe packet leads to a 1518 byte frame once the 18 byte ethernet header is
included.

    data_payload(ppp_mtu) + ppp_header(8) + eth_header(18) 
                    <= 
                            eth_mtu(1500) + eth_header(18)

| boost_mtu  | eth_mtu | ppp_mtu | notes                                      |
| ---------: | -------:| -------:| ------------------------------------------ |
| false      | 0       |    1492 | Standard setup                             |
| false      | 1500    |    1452 | Optimal for ATM connections                |
| false      | 1500    |    1432 | Permits non-default IP options             |
| true       | 1508    |    1500 | Uses jumbo frames - may not be accepted.   |

If your PPPoE link uses ATM as part of its connection to the remote network (likely, for DSL and
ADSL), another layer of packetisation and fragmentation is present. ATM uses 48 byte fixed length
packets, meaning that a lower MTU at the PPP level is actually better: 1452 byte PPP link packets
result in slightly lower overheads overall.

See https://www.sonicwall.com/support/knowledge-base/how-can-i-optimize-pppoe-connections/170505851231244/ 

```yaml
pppoe_ppp_mtu: 1500
```

The name of the systemd service used to manage pppd.

```yaml
pppoe_service_name: pppoe
```

The name used for the PPPd provider config. You could change this to reflect the ISP used,
for example.

```yaml
pppoe_call_name: provider
```

Basename of the created network adapter.

```yaml
pppoe_link_name: pppoe
```

## Dependencies

None.

## Example Playbook

    - hosts: drupalvm
      roles:
         - rivimey.pppoe

## License

MIT / BSD

## Author Information

This role was created in 2020 by Ruth Ivimey-Cook based on a posting on askubuntu by Robie Basak (https://askubuntu.com/users/7808/robie-basak) from 2016:

- https://askubuntu.com/questions/987982/how-do-i-use-netplan-to-configure-pppoe
