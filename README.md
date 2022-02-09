# openwrt-wg

```bash
vim /etc/init.d/refilliptableswg
```

```sh
#!/bin/sh
iptables -t mangle -F MARKFORWG
iptables -t mangle -A MARKFORWG -j RETURN
ips_file_path="/tmp/ip_addresses_to_wg.txt"
if [ -f "$ips_file_path" ]; then
    for ip_addr in $(cat ${ips_file_path} | grep -ve '^\s*$' | grep -ve '^\s*#' | uniq); do
        iptables -t mangle -I MARKFORWG --destination ${ip_addr} -j MARK --set-mark 0xc8/0xffffffff
    done
fi
```


```bash
chmod +x /etc/init.d/refilliptableswg
```

---

```bash
vim /etc/init.d/dwwgips
```

```sh
#!/bin/sh
echo "Run download list"
ips_file_path="/tmp/ip_addresses_to_wg.txt"
wget -O ${ips_file_path} https://raw.githubusercontent.com/revengel/openwrt-wg/main/ip_addresses_to_wg.txt
```

```bash
chmod +x /etc/init.d/dwwgips
```

---


```bash
vim /etc/firewall.user
```

```sh
# create chain
iptables -t mangle -N MARKFORWG
# jump to MARKFORWG
iptables -t mangle -A PREROUTING -j MARKFORWG
iptables -t mangle -A OUTPUT -j MARKFORWG
/etc/init.d/refilliptableswg
```

---

```sh
vim /etc/hotplug.d/iface/30-rknroute
```

```bash
#!/bin/sh
ip route add default dev wg0 table 200
```

---

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

---

```bash
vim /etc/init.d/ip2wg
```

```bash
#!/bin/sh

START=99

/etc/init.d/dwwgips
echo "Firewall restart"
/etc/init.d/firewall restart
```

```bash
chmod +x /etc/init.d/ip2wg
ln -s /etc/init.d/ip2wg /etc/rc.d/S99ip2wg
```

---

```bash
vim /etc/init.d/ip2wghot
```

```sh
#!/bin/sh

START=99

/etc/init.d/dwwgips
/etc/init.d/refilliptableswg
```

```bash
chmod +x /etc/init.d/ip2wghot
```

---

```bash
crontab -e
```

`0 */2 * * * /etc/init.d/ip2wghot`

```bash
/etc/init.d/cron enable
/etc/init.d/cron start
```
