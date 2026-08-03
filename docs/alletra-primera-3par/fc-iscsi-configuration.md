# FC / iSCSI Configuration

__Fibre Channel__

- Ensure proper zoning between hosts and array ports.

__iSCSI__

- Configure multipathd.
- Ensure initiators are registered on the array.
- Example config snippet in ```/etc/multipath.conf``` file:

  ```
  defaults {
      user_friendly_names yes
      find_multipaths yes
  }
  ```

__Notes__

- Multipathing ensures resiliency.

