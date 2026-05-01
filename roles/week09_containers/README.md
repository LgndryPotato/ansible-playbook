# week09_containers role

Implements Week 09 Containers scoring requirements:

- installs Docker safely with UT-approved bridge range `192.168.67.0/24`
- runs MinIO in Docker with a named volume
- exposes MinIO S3 through Apache HTTPS at `s3.<vm_name>.sysadm.ee`
- creates the `inventory-backup` bucket
- initializes and runs a Restic backup of `/data/inventory`
- containerises the Week 04 inventory API and stops the old systemd service

Add to `playbook.yml`:

```yaml
- role: week09_containers
  tags: week09_containers
```

Recommended vars:

```yaml
vm_name: "c11483vm"
vm_public_ip: "172.17.90.60"
```

Make sure OpenStack security group allows TCP 443 for the S3 HTTPS endpoint.
If you rerun Week 05 DNS later, also add this to the Week 05 forward zone template permanently:

```dns
s3             IN  A      {{ dns_effective_ip }}
```
