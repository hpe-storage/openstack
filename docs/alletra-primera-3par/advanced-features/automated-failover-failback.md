# Automated Failover (CFO) / Failback (CFB)

Cinder replication lets a replicated HPE Alletra MP backend be switched to its
secondary (DR) array when the primary is unavailable, and switched back once the
primary recovers. Run by hand, this means checking service state, picking the
right `backend_id`, issuing the Cinder command and then confirming that the
backend settled. Mistakes here can leave a backend half-switched.

Automated failover (CFO, Cinder failover) and failback (CFB, Cinder failback)
wrap that procedure in Ansible playbooks, so the same validated steps run every
time. The playbooks work with Peer Persistence (active-active) volumes and
groups, and run standalone or from an OpenStack-Ansible deployment.

## How it works

The playbooks do not talk to the storage arrays. They validate the request,
issue `cinder failover-host`, and watch the Cinder service record until the
backend reaches the expected replication status. The HPE Alletra MP Cinder
driver performs the array-side work.

1. **Preflight and capability check:** confirms the action and target are valid
   and that the deployed driver supports automated failback.
2. **Service and policy validation:** confirms the target cinder-volume service
   is `up`, not mid-transition, and that the requested `backend_id` is a
   configured `replication_device`.
3. **Idempotency check:** ends as a no-op if the backend is already in the
   requested state.
4. **Execution and settle polling:** sends the failover or failback request and
   polls the backend status until it completes, fails, or the settle window
   ends.
5. **Verification:** reports the final state of the target backend.

## Failover and failback at a glance

| | Failover (CFO) | Failback (CFB) |
|---|---|---|
| Purpose | Move the backend to the secondary (DR) array | Return the backend to the original primary array |
| Entry point | `cinder-failover.yml` | `cinder-failback.yml` |
| `backend_id` | The `replication_device` to fail over to | Always `default` |
| Works when primary is | Up (planned) or down (unplanned) | Up and reachable |
| Expected host status afterwards | `failed-over` | `enabled` |

## Prerequisites

- Replication is configured on the backend (`replication_device` is set).
- Every Peer Persistence remote copy group is started and fully synchronized.
- For failback, the original primary array is reachable and the DR remote copy
  groups are in sync.
- An OpenStack RC file with credentials allowed to list Cinder services and
  invoke backend failover.

## Example

Failover:

```bash
ansible-playbook -i local-inventory cinder-failover.yml \
  -e cinder_replication_openrc=/opt/stack/devstack/openrc \
  -e cinder_replication_target='<host>@<backend>' \
  -e cinder_replication_backend_id='<replication_device_backend_id>'
```

Failback:

```bash
ansible-playbook -i local-inventory cinder-failback.yml \
  -e cinder_replication_openrc=/opt/stack/devstack/openrc \
  -e cinder_replication_target='<host>@<backend>'
```

Run both commands from `alletramp/playbooks/openstack_cinder/`. After each run,
also check the volumes (`cinder list --all-projects`), because the host status
only shows that the backend finished, not that every volume switched.

## More information

For the full variable list, per-scenario driver behavior, result and status
reference, and troubleshooting, see the
<a href="https://github.hpe.com/hpe/hpe_storage_ansible_modules/blob/pike_changes/alletramp/playbooks/openstack_cinder/README.md"> Ansible Cinder Replication Component for CFO/CFB</a>.
