# Configure Keepalived

## Determine your IP address

Run the following command to find your IP address and interface in use on both primary and secondary servers.

```shell
ip -br a
```

???+ example "Example output"

    ```text
    lo               UNKNOWN        127.0.0.1/8 ::1/128
    ens192           UP             10.0.100.12/24 fe81::21c:24fe:fe2a:4c96/64
    ```

    The interface, for this example, is `ens192` with the ip `10.0.100.12`

Example values that will be used.

Server | Interface | IP
------ | --------- | --
Primary | eth0 | 10.0.100.11
Backup | ens192 | 10.0.100.12
VIP* | - | 10.0.100.10/24

<small>_\*the VIP is any unused IP space in your subnet. This example will use `10.0.100.10/24` (as shown in the above table)._</small>

## Configuration

Use the editor of your choice to open the keepalived configuration on both Primary and Backup servers. You can add as many backup servers using the backup server configuration.

```shell
sudo nano /etc/keepalived/keepalived.conf
```

### Primary Server <small>(Master)</small>

Change the values according to the following table.

Field | Current Value | Change to
----- | ------------- | ---------
interface | eth0 | Your Primary server's Interface.
mcast_src_ip | 10.0.100.11 | Your Primary server's IP.
auth_pass | xxXxxxXX | New 8 character password. (This will be the same in both configurations)
virtual_ipaddress | 10.0.100.10/24 | Your chosen VIP.

``` shell hl_lines="10 14 18 22" title="Primary Server: /etc/keepalived/keepalived.conf"
vrrp_track_process ftl {
    process "/usr/bin/pihole-FTL"
    delay 2
    quorum 1
    full_command true
}

vrrp_instance PIHOLE {
    state MASTER
    interface eth0
    virtual_router_id 55
    priority 150
    advert_int 1
    mcast_src_ip 10.0.100.11

    authentication {
        auth_type PASS
        auth_pass xxXxxxXX # (1)
    }

    virtual_ipaddress {
        10.0.100.10/24 # (2)
    }

    track_process {
        ftl
    }

}
```

1. `auth_pass` should match on all servers.
2. `virtual_ipaddress` should match on all servers.

### Backup Server(s)

Change the values according to the following table for each backup server.

Field | Current Value | Change to
----- | ------------- | ---------
interface | ens192 | Your Backup server's Interface.
mcast_src_ip | 10.0.100.12 | Your Backup server's IP.
auth_pass | xxXxxxXX | New 8 character password. (This will be the same in both configurations)
virtual_ipaddress | 10.0.100.10/24 | Your chosen VIP.
Priority | 145 | For each backup server the priority can be decreased effectively setting failover order.

``` shell hl_lines="10 12 14 18 22" title="Backup Server: /etc/keepalived/keepalived.conf"
vrrp_track_process ftl {
    process "/usr/bin/pihole-FTL"
    delay 2
    quorum 1
    full_command true
}

vrrp_instance PIHOLE {
    state BACKUP
    interface ens192
    virtual_router_id 55
    priority 145
    advert_int 1
    mcast_src_ip 10.0.100.12

    authentication {
        auth_type PASS
        auth_pass xxXxxxXX # (1)
    }

    virtual_ipaddress {
        10.0.100.10/24 # (2)
    }

    track_process {
        ftl
    }

}
```

1. `auth_pass` should match on all servers.
2. `virtual_ipaddress` should match on all servers.

## Restart Keepalived

### Ubuntu/Raspberry Pi OS

<small>_(using snap) see [Install Keepalived](../install-keepalived) with snap._</small>

```shell
sudo snap restart keepalived
```

To check the status run the following.

```shell
sudo snap logs keepalived
```

### Debian

```shell
sudo systemctl restart keepalived.service
```

To check the status run the following.

```shell
sudo systemctl status keepalived.service
```

--8<-- "includes/abbreviations.md"
