# Alibaba Cloud Linux Deployment

This document describes a production-oriented deployment path for this Astro blog on Alibaba Cloud Linux.

## Deployment layout

- `docker-compose.yml`: local validation, maps host port `8080` to the blog container.
- `docker-compose.prod.yml`: production deployment, uses a dedicated Nginx gateway container and keeps the app container on an internal network.
- `deploy/nginx/conf.d/blog.conf.template`: production reverse proxy template.
- `.env.prod.example`: production environment variable template.

## Why this layout

- The blog app container is not directly exposed to the public internet in production.
- A dedicated gateway container owns the public port and can later proxy more services.
- Additional sites or APIs can be added by extending the gateway configuration instead of rewriting the app container.

## Server preparation

1. Connect to the server and identify the OS version.

```bash
cat /etc/os-release
```

2. Install Docker and the Compose plugin.

Alibaba Cloud documents separate commands for Alibaba Cloud Linux 2 and Alibaba Cloud Linux 3. Docker also documents repository-based installation for RPM-based systems.

### Alibaba Cloud Linux 3 or 4

```bash
sudo rm -f /etc/yum.repos.d/docker*.repo
sudo wget -O /etc/yum.repos.d/docker-ce.repo http://mirrors.cloud.aliyuncs.com/docker-ce/linux/centos/docker-ce.repo
sudo sed -i 's|https://mirrors.aliyun.com|http://mirrors.cloud.aliyuncs.com|g' /etc/yum.repos.d/docker-ce.repo
sudo dnf -y install dnf-plugin-releasever-adapter --repo alinux3-plus
sudo dnf -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
docker --version
docker compose version
```

### Alibaba Cloud Linux 2

```bash
sudo rm -f /etc/yum.repos.d/docker*.repo
sudo wget -O /etc/yum.repos.d/docker-ce.repo http://mirrors.cloud.aliyuncs.com/docker-ce/linux/centos/docker-ce.repo
sudo sed -i 's|https://mirrors.aliyun.com|http://mirrors.cloud.aliyuncs.com|g' /etc/yum.repos.d/docker-ce.repo
sudo yum install -y yum-plugin-releasever-adapter --disablerepo='*' --enablerepo=plus
sudo yum -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
docker --version
docker compose version
```

3. Create an application directory.

```bash
sudo mkdir -p /srv/blog
sudo chown -R $USER:$USER /srv/blog
```

## Upload the project

Copy the project to the server, for example:

```bash
scp -r E:\blog root@YOUR_SERVER_IP:/srv/blog
```

Or push the code to Git and clone it on the server.

## Prepare production files

1. Enter the project directory.

```bash
cd /srv/blog
```

2. Create the production environment file.

```bash
cp .env.prod.example .env.prod
```

3. Edit `.env.prod` and set the real domain.

```env
SITE_URL=https://blog.example.cn
BLOG_DOMAIN=blog.example.cn
```

4. Render the Nginx gateway configuration from the template.

```bash
source .env.prod
envsubst '${BLOG_DOMAIN}' < deploy/nginx/conf.d/blog.conf.template > deploy/nginx/conf.d/blog.conf
```

## Start the production stack

```bash
docker compose --env-file .env.prod -f docker-compose.prod.yml up -d --build
docker compose -f docker-compose.prod.yml ps
docker compose -f docker-compose.prod.yml logs -f
```

## Security group and firewall

For the first public test, open only the ports you need:

- `22/TCP`: SSH, restricted to your own IP if possible
- `80/TCP`: HTTP
- `443/TCP`: reserve for HTTPS after the certificate is ready

Avoid relying on default security group rules without review.

## Verification

Before DNS is ready, test by public IP:

```text
http://YOUR_SERVER_IP
```

After the domain is ready and the DNS `A` record points to the server:

```text
http://blog.example.cn
```

## Updating the blog

Whenever you publish changes:

```bash
cd /srv/blog
docker compose --env-file .env.prod -f docker-compose.prod.yml up -d --build
```

## Next recommended step

After DNS is stable, add HTTPS termination at the gateway layer by mounting certificates or introducing an ACME-based certificate workflow.
