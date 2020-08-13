
PoC of some concept:

 * Only static hosts definition  (know clients)
 * Stateless - for HA (dhcpd.lease not used)
 * Support for /31 or /32 (on-link) addressing (255.255.255.255 for Windows clients with address with prefix /31)
 * Allow diffrent address (subnet) assigment based on classification:
   * For boot clients (PXE, UEFI): /30 assigments (private "netboot" subnet)
   * For other cases - "public" addresess
 * Dedicated ipxe instances with embed MAC address for NIC selection (for servers with more than one network card)
 * Faster iPXE address aquisition
 * No DNS support in DHCPD


This patch add new options for include directive:

```
	# Add all files from /etc/dhcp/host/ with extension .conf
	include "/etc/dhcp/host/*.conf"


	# Add all files from /etc/dhcp/host/ and sub folders with extension .conf
	include "/etc/dhcp/host/*/*.conf"
```

```
Include: total files collected 103590 in 58 ms
Include: total files processed: 103590 in 37063 ms

So... toooo long (add precompiled binary token cache for config file for speed up?)

But for my needs it's OK:

# cold fs cache
Include: total files collected 6000 in 5 ms
Include: total files processed: 6000 in 1557 ms

# with fs cache
Include: total files collected 6000 in 3 ms
Include: total files processed: 6000 in 255 ms

```

This patch adds new keyword for hosts declartion:

```
	match if <class_name>
```

For example:

```
# This option is only presend during net bootstrap
option arch-type code 93 = unsigned integer 16;

# Match "netboots" clients (but exclude iPXE which will use "normal" addressing
class "is-netboot" {
        match if exists arch-type and (not (option user-class="iPXE"));
}

# Definie two static mappings
# One for "public" network
host O000c290e741d  {
        hardware ethernet 00:0c:29:0e:74:1d;
        fixed-address 10.255.255.11;
        option routers 10.255.255.10;
        if option vendor-class-identifier="MSFT 5.0" {
                option subnet-mask 255.255.255.255;
        } else {
                option subnet-mask 255.255.255.254;
        }
}

# One for netboot network
host N000c290e741d {
	match if "is-netboot";
        hardware ethernet 00:0c:29:0e:74:1d;
        fixed-address 100.120.120.2;
        option routers 100.120.120.1;
        option subnet-mask 255.255.255.252;
}
```

Full example in etc/dhcpd.conf

```
./configure --prefix=/usr/local --with-randomdev=/dev/urandom --with-srv-conf-file=/etc/dhcp/dhcpd.conf --with-srv-lease-file=/var/lib/dhcp/dhcpd.leases --with-srv-pid-file=/run/dhcpd.pid --disable-dhcpv6 --disable-tracing --disable-failover 
```
