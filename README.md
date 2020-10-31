# PPPoE Service Role

Create a systemd service to manage a PPPoE connection on the target system.

## Requirements

The role uses the systemd, pppd, and pppoe packages, but otherwise has no dependencies.

## Role Variables

Available variables are listed below:

Network device created if the PPP negotiation succeeds. Leave blank to accept
the default name. If set here and the device already exists, pppd will exit.

```yaml
pppoe_ppp_dev: ppp0
```

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

Please note that increasing the MTU is not guaranteed to work, so do you will have to
test and check yourself.

```yaml
pppoe_eth_mtu: 1508
```

The MTU configured on the created pppoe link. Ideally this would match the MTU used on the
main network, to avoid fragmentation, but may be constrained by the configuration of the PPPoE
peer and remote network.

On pure Ethernet, the specified Ethernet max packet size of 1500 bytes allows for 1460 bytes of
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
ADSL links), another layer of packetisation and fragmentation is present. ATM uses 48 byte fixed
length packets, meaning that a lower MTU at the PPP level is actually better: 1452 byte PPP link
packets result in slightly lower overheads overall.

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

Tell PPPD to use remote DNS services on local system, once connected. Disable this if you
manage DNS in other ways (e.g. statically, or with your own server).
```yaml
pppoe_usepeerdns: true
```

Tell PPPD to enable use of IPv6 addressing on the created interface: "+ipv6"
Note: this won't happen unless the remote peer is configured to offer it as well.
```yaml
pppoe_enableipv6: true
```

Tell PPPD to set this interface as the default route once connected.  Disable if you need
to configure routes with more granularity.
```yaml
pppoe_setdefaultroute: true
```

Username and password to be set in chap-secrets. The defaults here assume the actual
value is set in your vault.yml file. Only one user is supported at present.

```yaml
pppoe_client_user: "{{ __pppoe_client_user }}"
pppoe_client_password: "{{ __pppoe_client_password }}"
```


## Dependencies

None.

## Example Playbook

    - hosts: gateway
      vars:
        pppoe_eth_dev: eth1
        pppoe_client_user: "me@myisp"
        pppoe_client_password: "verysecret"
      roles:
         - rivimey.pppoe

## License

MIT / BSD

## Author Information

This role was created in 2020 by Ruth Ivimey-Cook <ruth at ivimey.org>

The technique is based on a posting on AskUbuntu by Robie Basak (https://askubuntu.com/users/7808/robie-basak)
from 2016:

- https://askubuntu.com/questions/987982/how-do-i-use-netplan-to-configure-pppoe
