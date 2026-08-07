# Troubleshooting

## Common Issues

__Share creation fails__

  - Verify the HPE Alletra MP B10000 file service is enabled.
  - Check connectivity to the WSAPI endpoint (``hpealletra_wsapi_url``).
  - Ensure the configured user has sufficient permissions (``edit`` role).

__Access rules not working__

  - Verify network connectivity between the client and the array's file ports.
  - Check that the IP address in the access rule is correct.
  - Ensure the share type's ``squash_option`` is appropriate for your use case.

__Manage operation fails__

  - Clear all access rules from the backend share.
  - Verify the filesystem size is a multiple of 1 GiB.
  - Ensure the share type's ``reduce`` value matches the backend share.

Note

Set ``hpealletra_debug = True`` in the backend section of
``/etc/manila/manila.conf`` to enable HTTP debugging to the Alletra array when
diagnosing connectivity or API issues.
