# SSH Command & Port Forwarding Action 🔗

A powerful GitHub Action for executing SSH commands and
establishing secure port forwarding tunnels with support for jump hosts,
dynamic port allocation, and comprehensive authentication methods.

## ✨ Features

- 🚀 **Remote Command Execution** - Execute commands on remote servers
- 🔗 **Port Forwarding** - Local and remote port forwarding with dynamic allocation
- 🦘 **Jump Host Support** - Multi-hop SSH connections through bastion hosts
- 🔐 **Multiple Authentication** - Private keys, passwords, and SSH agent support
- 🌐 **Cross-Platform** - Works on Linux, macOS, and Windows runners
- 🛡️ **Security First** - Known hosts verification and secure key handling
- 📊 **Comprehensive Logging** - Detailed output for debugging and monitoring

## 🚀 Quick Start

### Execute Remote Command

```yaml
- name: Deploy Application
  uses: lexbritvin/ssh-action@v1
  with:
    host: your-server.com
    username: deploy
    private-key: ${{ secrets.SSH_PRIVATE_KEY }}
    command: |
      cd /var/www/app
      git pull origin main
      npm install --production
      pm2 restart app
```

### Local Port Forwarding

```yaml
- name: Forward Database Port
  uses: lexbritvin/ssh-action@v1
  with:
    host: database-server.com
    username: dbadmin
    private-key: ${{ secrets.SSH_PRIVATE_KEY }}
    local-forwards: "5432:localhost:5432"
```

### Remote Port Forwarding with Dynamic Port

```yaml
- name: Expose Local Service
  uses: lexbritvin/ssh-action@v1
  with:
    host: tunnel.example.com
    username: tunnel
    private-key: ${{ secrets.SSH_PRIVATE_KEY }}
    remote-forwards: "0:localhost:3000"
```

## 📖 Complete Usage Examples

### Multi-Service Port Forwarding

```yaml
name: Database Migration
on: [ push ]

jobs:
  migrate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Database Tunnels
        uses: lexbritvin/ssh-action@v1
        with:
          host: production-bastion.company.com
          username: devops
          private-key: ${{ secrets.PRODUCTION_SSH_KEY }}
          local-forwards: |
            5432:postgres-primary.internal:5432,
            6379:redis-cluster.internal:6379,
            3306:mysql-replica.internal:3306
          timeout: 60
          keep-alive: 30

      - name: Run Database Migration
        run: |
          # Now you can connect to localhost:5432, localhost:6379, localhost:3306
          npm run migrate:production
```

### Jump Host Configuration

```yaml
- name: Access Internal Server via Bastion
  uses: lexbritvin/ssh-action@v1
  with:
    host: internal-server.private
    username: admin
    private-key: ${{ secrets.SSH_PRIVATE_KEY }}
    jump-hosts: "bastion1.company.com:22,bastion2.company.com:2222"
    command: "systemctl status nginx"
```

### Secure Deployment Pipeline

```yaml
name: Production Deployment

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production

    steps:
      - uses: actions/checkout@v4

      - name: Deploy to Production
        uses: lexbritvin/ssh-action@v1
        with:
          host: ${{ secrets.PRODUCTION_HOST }}
          username: ${{ secrets.PRODUCTION_USER }}
          private-key: ${{ secrets.PRODUCTION_SSH_KEY }}
          known-hosts: ${{ secrets.KNOWN_HOSTS }}
          timeout: 120
          command: |
            set -e
            echo "🚀 Starting deployment..."

            # Backup current version
            sudo cp -r /var/www/app /var/www/app.backup.$(date +%Y%m%d_%H%M%S)

            # ...

            echo "✅ Deployment completed successfully!"

          post-command: |
            echo "🧹 Cleaning up old backups..."
            find /var/www -name "app.backup.*" -mtime +7 -delete
            echo "✅ Cleanup completed"
```

## 📋 Input Parameters

### Connection Settings

| Parameter  | Description     | Required | Default      |
|------------|-----------------|----------|--------------|
| `host`     | Target SSH host | Yes      | -            |
| `port`     | SSH port        | No       | `22`         |
| `username` | SSH username    | No       | Current user |

### Authentication

| Parameter          | Description                    | Required | Default |
|--------------------|--------------------------------|----------|---------|
| `private-key`      | SSH private key content        | No       | -       |
| `private-key-path` | Path to SSH private key file   | No       | -       |
| `password`         | SSH password (not recommended) | No       | -       |
| `known-hosts`      | SSH known_hosts content        | No       | -       |

### Port Forwarding

| Parameter         | Description                 | Required | Default |
|-------------------|-----------------------------|----------|---------|
| `local-forwards`  | Local port forwards (`-L`)  | No       | -       |
| `remote-forwards` | Remote port forwards (`-R`) | No       | -       |

### Advanced Options

| Parameter      | Description                    | Required | Default |
|----------------|--------------------------------|----------|---------|
| `jump-hosts`   | Comma-separated jump hosts     | No       | -       |
| `extra-flags`  | Additional SSH flags           | No       | -       |
| `command`      | Command to execute             | No       | -       |
| `post-command` | Cleanup command                | No       | -       |
| `timeout`      | Connection timeout (seconds)   | No       | `30`    |
| `keep-alive`   | Keep-alive interval (seconds)  | No       | `60`    |
| `dry-run`      | Show command without executing | No       | `false` |

## 📤 Outputs

| Output           | Description                                            |
|------------------|--------------------------------------------------------|
| `pid`            | Process ID of the SSH tunnel                           |
| `allocated-host` | Public host for remote forward (dynamic allocation)    |
| `allocated-port` | Allocated port for remote forward (dynamic allocation) |

## 🔧 Port Forwarding Formats

### Local Forwards (`-L`)

Forward local ports to remote destinations:

```yaml
local-forwards: "8080:web-server:80"                   # localhost:8080 → web-server:80
local-forwards: "127.0.0.1:8080:database:5432"         # 127.0.0.1:8080 → database:5432
local-forwards: "3000:localhost:3000,8080:nginx:80"    # Multiple forwards
```

### Remote Forwards (`-R`)

Forward remote ports to local destinations:

```yaml
remote-forwards: "8080:localhost:3000"          # remote:8080 → localhost:3000
remote-forwards: "0:localhost:3000"             # Dynamic port allocation
remote-forwards: "0.0.0.0:9000:localhost:9000"  # Bind to all interfaces
```

## 🛡️ Security Best Practices

### Use Secrets and Private Keys (Recommended)

```yaml
# Store your private key in GitHub Secrets
- name: Secure SSH Connection
  uses: lexbritvin/ssh-action@v1
  with:
    host: secure-server.com
    username: deploy
    private-key: ${{ secrets.SSH_PRIVATE_KEY }}
    known-hosts: ${{ secrets.KNOWN_HOSTS }}
```

## 🐛 Troubleshooting

### Debug Mode

Enable detailed logging and disable hosts checking for troubleshooting:

```yaml
- name: Debug SSH Connection
  uses: lexbritvin/ssh-action@v1
  with:
    host: problematic-server.com
    username: debug-user
    private-key: ${{ secrets.SSH_PRIVATE_KEY }}
    # Do not check a server key and use verbose SSH output
    extra-flags: "-o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -vvv"
    dry-run: true # Test command generation
```

## 📚 Additional Resources

- 📖 [SSH Port Forwarding Guide](https://www.ssh.com/academy/ssh/tunneling/example)
- 🔐 [SSH Key Management](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- 🛡️ [GitHub Secrets Management](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
- 🌐 [SSH Jump Host Configuration](https://www.redhat.com/sysadmin/ssh-proxy-bastion-proxyjump)

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

**⭐ Star this repo if you find it useful!**

Made with ❤️ for the GitHub Actions community
