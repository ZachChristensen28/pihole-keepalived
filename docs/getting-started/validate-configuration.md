# Validate Configuration

## Verify Keepalived

To verify keepalived is working as it should in the Master/backup states, the keepalived process can be temporarily disabled on the **Primary server**.

```shell
sudo systemctl stop keepalived.service

# - OR -

sudo snap stop keepalived
```

Verify that the **Backup server** has assumed the master state.

```shell
sudo systemctl status keepalived.service

# - OR -

sudo snap logs keepalived
```

???+ example "Example output from secondary server"

    ```shell
    Keepalived_vrrp[239456]: (PIHOLE) Backup received priority 0 advertisement
    Keepalived_vrrp[239456]: (PIHOLE) Entering MASTER STATE
    ```

If the backup goes into `MASTER STATE` then keepalived is working as intended.

Restart the keepalived service on the **Primary server**.

```shell
sudo systemctl start keepalived.service

# - OR -

sudo snap start keepalived
```

## Verify pihole-FTL failover

The [keepalived configuration](../configure-keepalived/) is setup to failover when the pihole-FTL process is no longer running. To test this configuration without restarting your server you can temporarily "break" the service.

!!! danger
    The following will temporarily break DNS resolution for the clients using the primary server.

On the **Primary server**, run the following command.

!!! danger "Stop the pihole process"
    Issue the following command to temporarily bring down the pihole-FTL service.

    ```shell
    sudo systemctl stop pihole-FTL.service
    ```

Check the status of Pihole to ensure DNS is not working.

```shell
pihole status
# [✗] DNS service is NOT running
```

Verify the **Backup server** has assumed the master state.

```shell
sudo systemctl status keepalived.service

# - OR -

sudo snap logs keepalived
```

???+ example "Example output from backup server"

    ```shell
    Keepalived_vrrp[239456]: (PIHOLE) Backup received priority 0 advertisement
    Keepalived_vrrp[239456]: (PIHOLE) Entering MASTER STATE
    ```

If the backup goes into `MASTER STATE` then keepalived process tracking is working as intended.

!!! check "Start the pihole process"

    ```shell
    sudo systemctl start pihole-FTL.service
    pihole status
    #   [✓] FTL is listening on port 53
    #      [✓] UDP (IPv4)
    #      [✓] TCP (IPv4)
    #      [✓] UDP (IPv6)
    #      [✓] TCP (IPv6)

    #   [✓] Pi-hole blocking is enabled
    ```
