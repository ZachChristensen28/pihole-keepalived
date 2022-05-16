# Configure Keepalived

## Create user for keepalived scripts

It is necessary to create a new system user for keeplaived to use to run tracking scripts. Otherwise, you can use an existing user or root.

``` shell title="Create keepalived_script user"
sudo useradd -r -s /sbin/nologin -M keepalived_script
```

## Create tracking script

A simple tracking script can be used to verify DNS resolution. For simplicity the script checks for the existence of TXT record for `pdc.ztsplunker.com`. This is a custom domain and it is recommended to switch this to a domain of your choosing.

Along with this script, the keepalived configuration will track the pihole-FTL process.

Save the following script in `/etc/keepalived/`.

``` shell title="check-dns.sh script"
#!/bin/bash
#
# Checks DNS resolution
#

RESULT=$(dig @localhost +time=1 +tries=1 +short {==pdc.ztsplunker.com==} txt)
test $? -ne 0 && exit 1
test -z $RESULT && exit 1
exit 0
```

- `pdc.ztsplunker.com` should be updated to a valid domain with the existence of a TXT record.

Set executable permissions for the script.

``` shell
sudo chmod +x /etc/keepalived/check-dns.sh
```

## Determine your IP address

Run the following command to find your IP address and interface in use on the keepalived servers.

```shell
ip -br a
```

???+ example "Example output"

    ```text
    lo               UNKNOWN        127.0.0.1/8 ::1/128
    {==ens192==}           UP             {==10.0.100.12==}/24 fe81::21c:24fe:fe2a:4c96/64
    ```

    The interface, for this example, is `ens192` with the ip `10.0.100.12`

Example values that will be used.

Server | Interface | IP | Priority
------ | --------- | -- |
Primary | eth0 | 10.0.100.11 | 150
Backup | ens192 | 10.0.100.12 | 145
VIP* | - | 10.0.100.10 | -

<small>_\*the VIP is any unused IP space in your subnet. This example will use `10.0.100.10` (as shown in the above table)._</small>

## Configuration

Use the editor of your choice to open the keepalived configuration on each keepalived server. You can add as many servers using the following configuration.

```shell
sudo nano /etc/keepalived/keepalived.conf
```

!!! note
    Each server will be configured as `BACKUP` allowing the highest priority to be set to the master.

### Primary Server <small>(Master)</small>

Change the values according to the following table.

Field | Current Value | Change to
----- | ------------- | ---------
interface | eth0 | Your Primary server's Interface.
Priority | 150 | (make sure the primary has the highest priority)
mcast_src_ip | 10.0.100.11 | Your Primary server's IP.
auth_pass | xxXxxxXX | New 8 character password (This will be the same across all servers).
virtual_ipaddress | 10.0.100.10 | Your chosen VIP.

``` shell title="Primary Server: /etc/keepalived/keepalived.conf"
global_defs{
    max_auto_priority 10
    enable_script_security
    script_user keepalived_script # (1)
}

vrrp_track_process ftl { # (4)
    process "/usr/bin/pihole-FTL"
    delay 2
    quorum 1
    full_command true
}

vrrp_script check_dns {
    script /etc/keepalived/check-dns.sh # (5)
    interval 4
    timeout 2
}

vrrp_instance PIHOLE {
    state BACKUP
    interface {==eth0==}
    virtual_router_id 100
    priority {==150==}
    advert_int 1
    mcast_src_ip {==10.0.100.11==}

    authentication {
        auth_type PASS
        auth_pass {==xxXxxxXX==} # (2)
    }

    virtual_ipaddress {
        {==10.0.100.10==} # (3)
    }

    track_process {
        ftl
    }

    track_script {
        check_dns
    }

}
```

1. `keepalived_script` user must be created before proceeding (see [Create user for keepalived scripts](#create-user-for-keepalived-scripts)).
2. `auth_pass` should match on all servers.
3. `virtual_ipaddress` should match on all servers.
4. Tracks the existence pihole-FTL process.
5. Confirm existence of tracking script and location (see [Create Tracking Script](#create-tracking-script)).

### Backup Server(s)

Change the values according to the following table for each backup server.

Field | Current Value | Change to
----- | ------------- | ---------
interface | ens192 | Your Backup server's Interface.
Priority | 145 | For each backup server the priority can be decreased effectively setting failover order.
mcast_src_ip | 10.0.100.12 | Your Backup server's IP.
auth_pass | xxXxxxXX | New 8 character password (This will be the same across all servers).
virtual_ipaddress | 10.0.100.10 | Your chosen VIP.

``` shell title="Backup Server: /etc/keepalived/keepalived.conf"
global_defs{
    max_auto_priority 10
    enable_script_security
    script_user keepalived_script # (1)
}

vrrp_track_process ftl { # (4)
    process "/usr/bin/pihole-FTL"
    delay 2
    quorum 1
    full_command true
}

vrrp_script check_dns {
    script /etc/keepalived/check-dns.sh # (5)
    interval 4
    timeout 2
}

vrrp_instance PIHOLE {
    state BACKUP
    interface {==ens192==}
    virtual_router_id 100
    priority {==145==}
    advert_int 1
    mcast_src_ip {==10.0.100.12==}

    authentication {
        auth_type PASS
        auth_pass {==xxXxxxXX==} # (2)
    }

    virtual_ipaddress {
        {==10.0.100.10==} # (3)
    }

    track_process {
        ftl
    }

    track_script {
        check_dns
    }

}
```

1. `keepalived_script` user must be created before proceeding (see [Create user for keepalived scripts](#create-user-for-keepalived-scripts)).
2. `auth_pass` should match on all servers.
3. `virtual_ipaddress` should match on all servers.
4. Tracks the existence pihole-FTL process.
5. Confirm existence of tracking script and location (see [Create Tracking Script](#create-tracking-script)).

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
