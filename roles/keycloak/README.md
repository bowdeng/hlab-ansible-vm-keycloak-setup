# Keycloak Ansible Role

This Ansible role installs and configures Keycloak with PostgreSQL database backend, HTTPS support, custom themes, and realm import.

## Features

- Installs Keycloak 23.0.7 with OpenJDK 17
- Configures PostgreSQL database backend
- Sets up HTTPS using provided SSL certificates
- Deploys custom Autolab theme
- Imports the Autolab realm configuration
- Configures Keycloak as a systemd service
- Health and metrics endpoints enabled

## Requirements

- Ubuntu/Debian-based system
- PostgreSQL installed (via postgresql role)
- SSL certificates in `files/certs/` directory:
  - `hlab_agentsofchaos_io.crt` - Server certificate
  - `hlab_agentsofchaos_io.pem` - Private key
  - `hlab_bundle-g2.crt` - CA bundle

## Role Variables

Key variables defined in `defaults/main.yml`:

```yaml
keycloak_version: "23.0.7"
keycloak_db_name: "keycloak"
keycloak_admin_user: "admin"
keycloak_hostname: "keycloak.hlab.agentsofchaos.io"
keycloak_https_port: "8443"
```

Override these in `group_vars/all`:

```yaml
keycloak_db_password: "YourSecurePassword"
keycloak_admin_password: "YourAdminPassword"
keycloak_keystore_password: "YourKeystorePassword"
```

## Directory Structure

```
roles/keycloak/
├── defaults/main.yml          # Default variables
├── handlers/main.yml          # Service handlers
├── tasks/main.yml             # Main tasks
├── templates/
│   ├── keycloak.service.j2    # Systemd service template
│   └── keycloak.conf.j2       # Keycloak configuration template
├── files/
│   ├── autolab.json           # Realm configuration
│   └── certs/
│       ├── hlab_agentsofchaos_io.crt
│       ├── hlab_agentsofchaos_io.pem
│       ├── hlab_bundle-g2.crt
│       └── autolab-theme/     # Custom theme files
└── README.md
```

## Usage

Include this role in your playbook:

```yaml
- name: Configure Keycloak Server
  hosts: keycloak_node
  become: yes
  roles:
    - common
    - postgresql
    - keycloak
```

## What the Role Does

1. **System Setup**
   - Installs Java 17 and required packages
   - Creates keycloak user and group
   - Creates installation directories

2. **Database Configuration**
   - Creates Keycloak database in PostgreSQL
   - Creates database user with appropriate privileges

3. **Keycloak Installation**
   - Downloads and extracts Keycloak
   - Configures database connection
   - Sets up hostname and network settings

4. **SSL/TLS Configuration**
   - Copies SSL certificates to Keycloak
   - Creates certificate chain
   - Generates PKCS12 keystore
   - Configures HTTPS on port 8443

5. **Theme Deployment**
   - Deploys Autolab custom theme
   - Configures theme directory structure

6. **Service Configuration**
   - Creates systemd service unit
   - Enables and starts Keycloak service
   - Configures automatic restart on failure

7. **Realm Import**
   - Imports Autolab realm from autolab.json
   - Applies realm configuration

## Accessing Keycloak

After successful deployment:

- **URL**: `https://keycloak.hlab.agentsofchaos.io:8443`
- **Admin Console**: `https://keycloak.hlab.agentsofchaos.io:8443/admin`
- **Admin Username**: Set in `keycloak_admin_user` (default: admin)
- **Realm**: autolab

## Troubleshooting

### Check Service Status
```bash
sudo systemctl status keycloak
```

### View Logs
```bash
sudo journalctl -u keycloak -f
```

### Verify Database Connection
```bash
sudo -u postgres psql -d keycloak -c "\dt"
```

### Test HTTPS Certificate
```bash
openssl s_client -connect keycloak.hlab.agentsofchaos.io:8443
```

## Security Considerations

- Change default passwords in `group_vars/all`
- Ensure SSL certificates are valid and up to date
- Restrict database access to localhost
- Use firewall rules to limit access to Keycloak ports
- Regularly update Keycloak version

## Dependencies

- `postgresql` role for database setup
- `common` role for basic system configuration

## License

MIT

## Author

Created for Autolab infrastructure
