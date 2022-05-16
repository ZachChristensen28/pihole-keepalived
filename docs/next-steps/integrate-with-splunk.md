# Integrate with Splunk

Track state changes using [Splunk HTTP Event Collector](https://docs.splunk.com/Documentation/Splunk/latest/Data/UsetheHTTPEventCollector) (HEC).

The following will create similar events in Splunk allowing you to more easily track state changes over time.

![Splunk HEC Event](../../images/splunk_hec_event.png)

## Prerequisites

See [Splunk Docs](https://docs.splunk.com/Documentation/Splunk/latest/Data/UsetheHTTPEventCollector#Configure_HTTP_Event_Collector_on_Splunk_Enterprise) on how to configure the HTTP Event Collector on Splunk Enterprise.

Save created HEC token for next steps.

## Create script

The VRRP instance can be configured to run a script when there is a state change. The following is an example script to be used with Splunk.

Place script in `/etc/keepalived`.

``` text title="notify-splunk.sh"
#!/bin/bash
#
# Send state changes to Splunk
#
hostname=$(hostname)
state=$3
cdate=$(date +%s)
data_gen(){
        cat <<EOF
{
  "time": "$cdate",
  "host": "$hostname",
  "source": "${hostname}-keepalived-hec",
  "sourcetype": "_json",
  {=="index": "netops",==}
  "event": {"event": "state-change", "state": "$state"}
}
EOF
}

curl -k "https://{==splunk-ip-fqdn:8088==}/services/collector/event" \
        -H "Authorization: Splunk {==$SPLUNK_HEC_TOKEN==}" \
        -d "$(data_gen)"
```

- The `index` field in the above script is hard coded to `netops`. This may be removed if you want default to the index that was sent upon the creation of the HEC token. Or replace with the desired index.
- Update the curl command with the correct IP or FQDN.
- `SPLUNK_HEC_TOKEN` may be replaced with the actual token value. For better script security, an environment variable can be used.

Give the script executable permissions when finished.

```shell
sudo chmod +x /etc/keepalived/notify-splunk.sh
```

## Update keepalived.conf

Add `notify /etc/keepalived/notify-splunk.sh` to the vrrp_instance.

```shell title="keepalived.conf example"
...

vrrp_instance PIHOLE {
    state BACKUP
    interface eth0
    virtual_router_id 100
    priority 150
    advert_int 1
    mcast_src_ip 10.0.10.10
    {++notify /etc/keepalived/notify-splunk.sh++}

...
```

Restart the keepalived service.

--8<-- "includes/abbreviations.md"
