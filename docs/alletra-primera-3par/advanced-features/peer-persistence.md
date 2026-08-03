# Peer Persistence (Classic PP only)

Given 3PAR/Primera backend configured with replication setup, currently only Active/Passive replication is supported by 3PAR/Primera in OpenStack. When failover happens, nova does not support volume force-detach (from dead primary backend) / re-attach to secondary backend. Storage engineer’s manual intervention is required.

To overcome above scenario, support for Peer Persistence is added. Given a system with Peer Persistence configured and replicated volume is created. When this volume is attached to an instance, vlun is created automatically in secondary backend, in addition to primary backend. So that when a failover happens, it is seamless.

For Peer Persistence support, perform following steps: 1] enable multipath 2] set replication mode as “sync” 3] configure a quorum witness server

Specify ip address of quorum witness server in ``/etc/cinder/cinder.conf`` [within backend section] as given below:

```
[3pariscsirep]
hpe3par_api_url = http://10.50.3.7:8008/api/v1
hpe3par_username = <user_name>
hpe3par_password = <password>
...
<other parameters>
...
replication_device = backend_id:CSIM-EOS12_1611702,
                     replication_mode:sync,
                     quorum_witness_ip:10.50.3.192,
                     hpe3par_api_url:http://10.50.3.22:8008/api/v1,
                     ...
                     <other parameters>
                     ...
```

