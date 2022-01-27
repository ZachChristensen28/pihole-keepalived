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

    The interface I would use for this example is `ens192` with the ip `10.0.100.12`

Example values that will be used.

Server | Interface | IP
------ | --------- | --
Primary | eth0 | 10.0.100.11
Secondary | ens192 | 10.0.100.12
VIP* | - | 10.0.100.10/24

<small>_\*the VIP is any unused IP space in your subnet. This example will use `10.0.100.10/24`._</small>

## Configuration

Use the editor of your choice to open the keepalived configuration on both Primary and Secondary servers.

```shell
sudo nano /etc/keepalived/keepalived.conf
```

### First Node

Change the values according to the following table.

Field | Current Value | Change to
----- | ------------- | ---------
interface | eth0 | Primary Interface
unicast_src_ip | 10.0.100.11 | Your Primary Server IP
unicast_peer | 10.0.100.12 | Your Secondary Server IP
auth_pass | xxXxxxXX | New 8 character password. (This will be the same in both configurations)
virtual_ipaddress | 10.0.100.10/24 | Your chosen VIP

```text
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
    unicast_src_ip 10.0.100.11
    unicast_peer {
        10.0.100.12
    }

    authentication {
        auth_type PASS
        auth_pass xxXxxxXX
    }

    virtual_ipaddress {
        10.0.100.10/24
    }

    track_process {
        ftl
    }

}
```

### Second Node

Change the values according to the following table.

Field | Current Value | Change to
----- | ------------- | ---------
interface | ens192 | Secondary Interface
unicast_src_ip | 10.0.100.12 | Your Secondary Server IP
unicast_peer | 10.0.100.11 | Your Primary Server IP
auth_pass | xxXxxxXX | New 8 character password. (This will be the same in both configurations)
virtual_ipaddress | 10.0.100.10/24 | Your chosen VIP

```text
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
    unicast_src_ip 10.0.100.12
    unicast_peer {
        10.0.100.11
    }

    authentication {
        auth_type PASS
        auth_pass xxXxxxXX
    }

    virtual_ipaddress {
        10.0.100.10/24
    }

    track_process {
        ftl
    }

}
```

--8<-- "includes/abbreviations.md"

## Restart Keepalived

### Ubuntu

<small>_(using snap) see [Install Keepalived](../install-keepalived/#ubuntu) with snap._</small>

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
