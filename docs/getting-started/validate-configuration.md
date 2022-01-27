# Validate Configuration

## Verify Keepalived

To verify keepalived is working as it should in the Master/backup states, the keepalived process can be temporarily disabled on the **Primary server**.

```shell
sudo systemctl stop keepalived.service

# - OR -

sudo snap stop keepalived # on Ubuntu using snap
```

Verify that the **Secondary server** has assumed the master state.

```shell
sudo systemctl status keepalived.service

# - OR -

sudo snap logs keepalived # on Ubuntu using snap
```

???+ example "Example output from secondary server"

    ```shell
    Keepalived_vrrp[239456]: (PIHOLE) Backup received priority 0 advertisement
    Keepalived_vrrp[239456]: (PIHOLE) Entering MASTER STATE
    ```

If the secondary goes into `MASTER STATE` then keepalived is working as intended.

Restart the keepalived service on the **Primary server**.

```shell
sudo systemctl start keepalived.service

# - OR -

sudo snap start keepalived # on Ubuntu using snap
```

## Verify pihole-FTL failover

The [keepalived configuration](../configure-keepalived/) is setup to failover when the pihole-FTL process is no longer running. To test this configuration without restarting your server you can temporarily "break" the service.

!!! danger
    The following will temporarily break DNS resolution for the clients using the primary server.

On the **Primary server**, run the following command.

<small>_(This command will not break the pihole-FTL service until we issue a restart command of pihole)_</small>

```shell
echo "BREAK" | sudo tee /etc/dnsmasq.d/99-failover-test.conf
```

!!! danger "Restart the pihole process"
    Issue the following command to temporarily bring down the pihole-FTL service.

    ```shell
    pihole restartdns
    ```

Check the status of Pihole to ensure DNS is not working.

```shell
pihole status
# [✗] DNS service is NOT running
```

Verify the **Secondary server** has assumed the master state.

```shell
sudo systemctl status keepalived.service

# - OR -

sudo snap logs keepalived # on Ubuntu using snap
```

???+ example "Example output from secondary server"

    ```shell
    Keepalived_vrrp[239456]: (PIHOLE) Backup received priority 0 advertisement
    Keepalived_vrrp[239456]: (PIHOLE) Entering MASTER STATE
    ```

If the secondary goes into `MASTER STATE` then keepalived process tracking is working as intended.

Remove the previously created file and restart the pihole service to resume DNS functionality.

```shell
sudo rm /etc/dnsmasq.d/99-failover-test.conf
pihole restartdns
#   [✓] Restarting DNS server
pihole status
#   [✓] FTL is listening on port 53
#      [✓] UDP (IPv4)
#      [✓] TCP (IPv4)
#      [✓] UDP (IPv6)
#      [✓] TCP (IPv6)

#   [✓] Pi-hole blocking is enabled
```
