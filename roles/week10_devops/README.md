# week10_devops

Ansible role for Sysadmin Week 10: DevOps.

## What it does

- Creates `/home/centos/lab10`.
- Installs Dive, Trivy, and `docker-compose-plugin`.
- Pulls `registry.hpc.ut.ee/public/lab10-consultancy:latest` without running it.
- Deploys `/home/centos/lab10/docker-compose.yml` for MinIO and the inventory API.
- Declares the existing MinIO Docker volume as `external: true` so Week 9 backup data is not replaced.
- Validates the Compose file with `docker compose config`.

By default it does **not** stop/remove old containers or run `docker compose up -d`, because the lab page recommends deploying the supporting files and running the migration manually once.

## Add to `playbook.yml`

```yaml
- hosts: vm1
  become: true
  roles:
    - week10_devops
```

## Manual audit commands

```bash
sudo dive registry.hpc.ut.ee/public/lab10-consultancy:latest
sudo trivy image --severity CRITICAL --no-progress registry.hpc.ut.ee/public/lab10-consultancy:latest
```

Write the findings:

```bash
echo 'sha256:<digest-or-12+-hex-prefix>' | sudo tee /home/centos/lab10/malicious_layer.txt
echo 'CVE-YYYY-NNNNN' | sudo tee /home/centos/lab10/malicious_cve.txt
```

Or set these variables before running Ansible again:

```yaml
week10_malicious_layer_digest: "sha256:..."
week10_malicious_cve_id: "CVE-YYYY-NNNNN"
```

## Migration commands after Ansible deploys the Compose file

First check the config and the old volume:

```bash
cd /home/centos/lab10
sudo docker compose config
sudo docker volume ls
sudo docker ps
```

Then migrate:

```bash
sudo docker stop minio inventory-api
sudo docker rm minio inventory-api
sudo docker compose -p lab10 up -d
sudo docker compose -p lab10 ps
```

Verify:

```bash
curl -I http://127.0.0.1:9000/minio/health/live
sudo docker inspect minio --format '{{ index .Config.Labels "com.docker.compose.project" }}'
sudo docker inspect inventory-api --format '{{ index .Config.Labels "com.docker.compose.project" }}'
```

## Optional full migration from Ansible

Set this only after you have verified that the variables match your Week 9 container setup:

```yaml
week10_manage_compose_stack: true
```
