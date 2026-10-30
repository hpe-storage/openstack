# Automated Failover (CFO) / Failback (CFB) Replication Recovery

Cinder replication lets a replicated HPE Alletra MP backend be switched to its
secondary (DR) array when the primary is unavailable, and switched back once the
primary recovers. Run by hand, this means checking service state, picking the
right `backend_id`, issuing the Cinder command and then confirming that the
backend settled. Mistakes during manual execution can leave a backend in an
intermediate or inconsistent failover state.

Automated failover (CFO, Cinder failover) and failback (CFB, Cinder failback)
wrap that procedure in Ansible playbooks, so the same validated steps run every
time. The playbooks work with Peer Persistence (active-active) volumes and
groups, and run standalone or from an OpenStack-Ansible deployment.

**How it works**

The playbooks do not talk to the storage arrays. They validate the request,
issue `cinder failover-host`, and watch the Cinder service record until the
backend reaches the expected replication status. The HPE Alletra MP Cinder
driver performs the array-side work.

1. **Preflight and capability check:** confirms the action and target are valid
   and that the deployed HPE Alletra MP Cinder driver advertises support
   (`AUTO_FAILBACK_SUPPORTED`) for failover and failback through the Cinder
   `failover-host` API. If it does not, the run ends as a no-op.
2. **Service and policy validation:** confirms the target cinder-volume service
   is `up` and not mid-transition (`replication_status` is not
   `failing-over`). For failover, `replication_status` must also be `enabled`
   or `failed-over`. It then confirms the requested `backend_id` is a
   configured `replication_device`.
3. **Idempotency check:** ends as a no-op if the backend is already in the
   requested state.
4. **Execution and settle polling:** sends the failover or failback request and
   polls the backend status until it completes, fails, or the settle window
   ends. During polling, the playbooks monitor the service
   `replication_status` and wait until the backend reaches the expected
   status (`failed-over` after failover, `enabled` after failback). A backend
   still `failing-over` when the window ends is reported as a warning.
5. **Verification:** reports the final state of the target backend.

**Failover and failback at a glance**

| Item | Failover (CFO) | Failback (CFB) |
|---|---|---|
| Purpose | Move backend to secondary (DR) array | Return backend to primary array |
| Entry point | `cinder-failover.yml` | `cinder-failback.yml` |
| `backend_id` | Target `replication_device` backend | `default` |
| Primary array state | Available (planned failover) or unavailable (disaster recovery failover) | Available and reachable |
| Expected `replication_status` | `failed-over` | `enabled` |

For failback, Cinder uses `backend_id=default`, which instructs the driver to
return to the original primary backend.

**Prerequisites**

- Replication is configured on the backend (`replication_device` is set).
- Every Peer Persistence remote copy group is started and fully synchronized.
- For failback, the original primary array is reachable and the DR remote copy
  groups are in sync.
- An OpenStack RC file with credentials allowed to list Cinder services and
  invoke backend failover.
- The playbook runs on the cinder-volume host, so the capability check can
  import the driver and `cinder.conf` can be read.

**Example**

Failover:

```bash
ansible-playbook -i inventories/local-inventory cinder-failover.yml \
  -e cinder_replication_openrc=/opt/stack/devstack/openrc \
  -e cinder_replication_target='<host>@<backend>' \
  -e cinder_replication_backend_id='<replication_device_backend_id>'
```

Failback:

```bash
ansible-playbook -i inventories/local-inventory cinder-failback.yml \
  -e cinder_replication_openrc=/opt/stack/devstack/openrc \
  -e cinder_replication_target='<host>@<backend>'
```

Run both commands from `alletramp/playbooks/openstack_cinder/`. After each run,
check the volumes as well:

```bash
cinder service-list --withreplication
cinder list --all-projects --field ID,Name,Status,os-vol-host-attr:host,replication_status
```

The backend status only indicates that the failover or failback workflow
completed. If individual volumes fail, the backend can still end `failed-over`
or `enabled` while those volumes show `replication_status` `error`. Verify that
replicated volumes and consistency groups report the expected replication
status and backend assignment.

**Limitations**

- The playbooks orchestrate Cinder replication operations and do not perform
  storage failover directly.
- Successful backend failover does not guarantee application-level
  availability.
- Network connectivity between OpenStack services and the target backend must
  be available.
- A request the driver rejects, such as a remote copy group that is not started
  or a primary array that is still down on failback, leaves the backend
  unchanged. The playbook reports the failure only after the full settle
  window.

**More information**

For the full variable list, driver scenarios and status rules, result and status
reference, and troubleshooting, see the
[Ansible Cinder Replication Component for CFO/CFB][cinder-readme].

[cinder-readme]: ../README.md
