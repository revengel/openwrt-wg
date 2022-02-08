# openwrt-wg

```bash
vim /etc/firewall.user
```

```bash
# create chain
iptables -t mangle -N MARKFORWG
iptables -t mangle -F MARKFORWG
iptables -t mangle -A MARKFORWG -j RETURN

# jump to MARKFORWG
iptables -t mangle -A PREROUTING -j MARKFORWG
iptables -t mangle -A OUTPUT -j MARKFORWG

ips_file_path="/tmp/ip_addresses_to_wg.txt"
if [ -f "$ips_file_path" ]; then
    for ip_addr in $(cat ${ips_file_path} | grep -ve '^\s*$' | grep -ve '^\s*#' | uniq); do
        iptables -t mangle -I MARKFORWG --destination ${ip_addr} -j MARK --set-mark 0xc8/0xffffffff
    done
fi
```

```bash
vim /etc/hotplug.d/iface/30-rknroute
```

```bash
#!/bin/sh
ip route add default dev wg0 table 200
```

```bash
vim /etc/config/network
```

```bash
# add to /etc/config/network
config rule
    option priority '100'
    option lookup '200'
    option mark '0xc8'
```

```bash
vim /etc/init.d/ip2wg
```

```bash
#!/bin/sh

START=99

echo "Run download list"
wget -O /tmp/ip_addresses_to_wg.txt https://raw.githubusercontent.com/revengel/openwrt-wg/main/ip_addresses_to_wg.txt
echo "Firewall restart"
/etc/init.d/firewall restart
```

```bash
chmod +x /etc/init.d/ip2wg
ln -s /etc/init.d/ip2wg /etc/rc.d/S99ip2wg
```

```bash
crontab -e
```

`0 4 * * * /etc/init.d/ip2wg`

```bash
/etc/init.d/cron enable
/etc/init.d/cron start
```
