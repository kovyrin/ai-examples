## Relevant Files

- `example-forum-deploy/.kamal/secrets` - Kamal secrets configuration
- `example-forum-deploy/config/deploy.yml` - Main Kamal deployment configuration
- `example-forum-deploy/Dockerfile` - Docker container definition for phpBB
- `example-forum-deploy/env.example.txt` - Environment variables template
- `example-forum-deploy/.gitignore` - Git ignore rules for sensitive files
- `example-forum-deploy/docker-compose.yml` - Local development/testing compose file
- `example-forum-deploy/scripts/backup.sh` - Automated backup script
- `example-forum-deploy/scripts/restore.sh` - Database restore script
- `example-forum-deploy/scripts/seed_preserved_files.sh` - Script to copy avatars/files to server
- `example-forum-deploy/config/php.ini` - PHP configuration overrides
- `example-forum-deploy/config/vhost.conf` - Apache vhost configuration for phpBB
- `example-forum-deploy/scripts/entrypoint.sh` - Container entrypoint script (supports custom commands)
- `example-forum-deploy/scripts/pre-deploy-check.sh` - Pre-deployment validation script
- `example-forum-deploy/phpbb/up.php` - Health check endpoint for kamal-proxy
- `example-forum-deploy/phpbb/healthcheck.php` - Health check for Docker container
- `example-forum-deploy/phpbb/reset_admin_password.php` - Admin password reset utility
- `example-forum-deploy/README.md` - Deployment documentation

#### Notes on Backup Implementation

- Backups are stored on the host at `/backups/example-forum/`
- Centralized backup location allows easy sync to NAS for all projects
- Volume mount ensures backups persist across container restarts
- Backup script uses web container environment variables (PHPBB*DATABASE*\*)
- Each backup is ~27MB compressed (from 57MB database + files)
- Backups are created atomically using /tmp to avoid syncing partial files
- Syncthing provides automated one-way replication from server to NAS
- Access Syncthing UI via Tailscale VPN: `http://server01:8384/`
- Alternative access via SSH tunnel: `ssh -L 8384:127.0.0.1:8384 ubuntu@server01.example.com`
- Daily backups run at 3:00 AM UTC via cron job in `/etc/cron.d/example-forum-backup`
- Backup logs stored in `/var/log/example-forum-backup/` with 30-day retention

## Notes

- This deployment uses Kamal for orchestration on Oracle Cloud Infrastructure
- Target server: ubuntu@server01.example.com (Ubuntu 24.04 ARM64)
- Memory constraints: 2GB container limit on 6GB VPS
- Database: MariaDB 11 as Kamal accessory service (no public port due to conflict)
- Initial deployment includes data migration from local Docker backup (25MB)
- Daily backups with 7 daily, 12 monthly, infinite annual retention
- Email delivery via Oracle Cloud Infrastructure Email Delivery service (configured and working)
- SSL via Let's Encrypt with kamal-proxy handling certificates (auto-renewal)
- Manual deployments from local machine (no CI/CD)
- Forum is currently accessible at https://example-forum.com (HTTPS with auto-redirect)
- Admin credentials: admin_user (password in secrets file)

### Implementation Guidelines

**ARM64 Compatibility:**

- Ensure all Docker images specify ARM64 variants (e.g., `mariadb:11` for ARM64)
- Test builds locally if you have an ARM64 machine or use Docker buildx for cross-platform builds

**Kamal Commands Reference:**

- `kamal init` - Initialize Kamal configuration
- `kamal setup` - First-time server setup (installs Docker, configures proxy)
- `kamal deploy` - Deploy application (build, push, pull, start containers)
- `kamal rollback [version]` - Rollback to previous version
- `kamal app logs` - View application logs
- `kamal app exec --interactive '<command>'` - Run commands in the container
- `kamal accessory boot [name]` - Start accessory service
- `kamal secrets push` - Push secrets to server
- `kamal shell` - Open interactive shell in container (alias)
- `kamal backup` - Create forum backup (alias)
- `kamal backup-cleanup` - Run backup retention cleanup (alias)

**Security Considerations:**

- Never commit `.kamal/secrets` or `config.php` to git
- Use strong passwords for database and admin accounts
- Configure firewall rules on Oracle Cloud to restrict access
- Regularly update Docker images for security patches
- Remove `reset_admin_password.php` after securing admin accounts

**phpBB Specific Configuration:**

- Set `$dbhost = 'example-forum-db'` in config.php (Kamal accessory name)
- Ensure `cache/`, `files/`, and `store/` directories are writable
- Configure email settings through ACP after initial deployment
- Test file uploads and avatar functionality
- Preserved files from backup have been seeded to `/home/ubuntu/example-forum/`

**SSL Configuration:**

- SSL is handled automatically by kamal-proxy using Let's Encrypt
- No Cloudflare needed - kamal-proxy manages certificates and auto-renewal
- HTTP automatically redirects to HTTPS
- Certificate valid for 90 days with automatic renewal before expiration

**Email Configuration:**

- Email delivery configured through Oracle Cloud Infrastructure Email Delivery
- SMTP settings configured in phpBB Admin Control Panel
- SPF and DKIM records set up for example-forum.com domain
- Approved sender domain configured in Oracle Cloud console
- Email functionality verified with password reset tests

**Backup and Recovery:**

- Test restore procedures before going live
- Verify backup integrity with test restores
- Store backup encryption keys securely
- Consider off-site backup storage for disaster recovery
- Use `scripts/seed_preserved_files.sh` to restore user uploads

**Deployment Workflow:**

1. Make changes locally in `example-forum-deploy` directory
2. Commit to git repository
3. Run `kamal deploy` from local machine
4. Monitor deployment with `kamal app logs`
5. Rollback if issues: `kamal rollback`

**Troubleshooting:**

- Check logs: `kamal app logs`, `kamal accessory logs db`
- SSH to server: `ssh ubuntu@server01.example.com`
- Container inspection: `docker ps`, `docker logs [container]`
- Kamal lock issues: `kamal lock release --force`
- Health check issues: kamal-proxy expects `/up.php` endpoint returning 200 OK
- Missing files: Ensure all phpBB config files are present (we had to add several)

**Performance Monitoring:**

- Monitor memory usage: `docker stats`
- Check disk space for backups: `df -h /home/ubuntu/example-forum/`
- Database performance: MySQL slow query log
- Application metrics: phpBB's built-in statistics

## Tasks

- [x] 1. Repository Setup and Structure

  - [x] 1.1. Create new directory `example-forum-deploy` within the project root

  - [x] 1.2. Initialize git repository in `example-forum-deploy` directory

  - [x] 1.3. Create directory structure: `phpbb/`, `config/`, `scripts/`, `hooks/`, `database/`

  - [x] 1.4. Copy phpBB files from `phpbb-recovery/phpbb_files/` to `example-forum-deploy/phpbb/`, excluding `config.php`

  - [x] 1.5. Copy database dump from local Docker container to `example-forum-deploy/database/dump/phpbb_forum_database.sql`

  - [x] 1.6. Create `.gitignore` file with rules for secrets, `phpbb/config.php`, and cache directories

  - [x] 1.7. Create `env.example.txt` template with required environment variables

  - [x] 1.8. Create initial README.md with deployment instructions

- [x] 2. Oracle Cloud Infrastructure Preparation

  - [x] 2.1. Test SSH access to ubuntu@server01.example.com

  - [x] 2.2. Verify server specifications (2 OCPU, 6GB RAM, Ubuntu 24.04 ARM64)

  - [x] 2.3. Configure firewall to allow port 80 (HTTP) and 443 (HTTPS) for incoming traffic

  - [x] 2.4. Create `/home/ubuntu/example-forum/` directory structure on the server

  - [x] 2.5. Verify Docker is pre-installed

  - [x] 2.6. Add temporary entry to local `/etc/hosts` for example-forum.com pointing to server IP

- [x] 3. Kamal Configuration

  - [x] 3.1. Install Kamal gem locally: `gem install kamal`

  - [x] 3.2. Run `kamal init` in `example-forum-deploy` directory

  - [x] 3.3. Configure `config/deploy.yml` with service name, image, servers, and registry settings

  - [x] 3.4. Configure persistent volumes in `deploy.yml` for phpBB (`files/`, `images/`, `store/`) and MariaDB data

  - [x] 3.5. Set up MariaDB accessory service configuration in deploy.yml

  - [x] 3.6. Configure environment variables for phpBB database connection

  - [x] 3.7. Set memory limits (2GB) in deploy.yml for the application container

  - [x] 3.8. Create `.kamal/secrets` file with registry password and database credentials

  - [x] 3.9. Configure kamal-proxy settings for SSL and domain handling

- [x] 4. phpBB Application Containerization

  - [x] 4.1. Create Dockerfile based on PHP 8.2 Apache ARM64 image

  - [x] 4.2. Create an entrypoint script that generates `config.php` from environment variables on container startup

  - [x] 4.3. Configure PHP extensions required by phpBB (mysqli, gd, xml, mbstring, etc.)

  - [x] 4.4. Create `config/php.ini` with memory_limit=2G and other optimizations

  - [x] 4.5. Set up Apache configuration (`config/vhost.conf`) for phpBB with proper rewrite rules

  - [x] 4.6. Configure file permissions for phpBB directories (cache, files, store)

  - [x] 4.7. Add health check endpoint (`phpbb/up.php`) for kamal-proxy

  - [x] 4.8. Create `phpbb/healthcheck.php` for Docker container health checks

  - [x] 4.9. Add missing phpBB config files that were causing 500 errors

- [x] 5. Initial Deployment and Validation

  - [x] 5.1. Run `kamal setup` to initialize server (Docker already installed)

  - [x] 5.2. Deploy MariaDB accessory with `kamal accessory boot db`

  - [x] 5.3. Database automatically restored from dump via MariaDB initialization

  - [x] 5.4. Deploy phpBB application with `kamal deploy`

  - [x] 5.5. Fix deployment issues (missing config.php files in phpBB)

  - [x] 5.6. Verify forum loads at http://example-forum.com (via /etc/hosts)

  - [x] 5.7. Confirm phpBB returns 200 OK and displays "К.А.М.П. - Index page"

  - [x] 5.8. Test user login with existing admin credentials (admin_user)

  - [x] 5.9. Fix broken avatars by seeding preserved files from backup

  - [x] 5.10. Verify Prosilver Dark theme displays correctly

  - [x] 5.11. Update DNS A record for example-forum.com to point to server (140.238.144.221)

  - [x] 5.12. Enable SSL in deploy.yml and redeploy for HTTPS (Let's Encrypt via kamal-proxy)

  - [x] 5.13. Perform final validation of https://example-forum.com (SSL working, auto-redirect enabled)

- [x] 6. Database Setup and Migration

  - [x] 6.1. Configure MariaDB 11 accessory in Kamal with ARM64 image

  - [x] 6.2. Set database environment variables (MYSQL_ROOT_PASSWORD, MYSQL_DATABASE)

  - [x] 6.3. Configure MariaDB without public port (due to conflict with existing MySQL)

  - [x] 6.4. Use local Docker database dump (25MB) instead of recovery file

  - [x] 6.5. Verify database connection from phpBB container

  - [x] 6.6. Create admin password reset script for account recovery

  - [x] 6.7. Document password reset process with detailed guide and quick reference

- [x] 7. Email Configuration

  - [x] 7.1. Set up Oracle Cloud Infrastructure Email Delivery service

  - [x] 7.2. Generate SMTP credentials in Oracle Cloud console

  - [x] 7.3. Configure approved sender domain (example-forum.com)

  - [x] 7.4. Add email configuration environment variables to Kamal secrets

  - [x] 7.5. Create phpBB email configuration with SMTP settings

  - [x] 7.6. Set up SPF records for example-forum.com domain

  - [x] 7.7. Configure DKIM signing for email authentication

  - [x] 7.8. Test email functionality with password reset

- [x] 8. Backup System Implementation

  - [x] 8.1. Create `/backups/example-forum/` directory structure on the server with proper permissions

  - [x] 8.2. Create `scripts/backup.sh` with the following functionality:

    - [x] 8.2.1. Generate timestamp-based directory name (YYYYMMDDHHMMSS-example-forum)
    - [x] 8.2.2. Create backup directory `/backups/example-forum/YYYYMMDDHHMMSS-example-forum/`
    - [x] 8.2.3. Dump MariaDB database to `database.sql` using mysqldump with UTF-8 handling:
      - Use `--default-character-set=utf8mb4` flag
      - Include `--single-transaction` for consistency
      - Add `--routines --triggers` to backup stored procedures
      - Set `--complete-insert` for better compatibility
      - Use `--hex-blob` for binary data safety
    - [x] 8.2.4. Copy phpBB files (`/files/`, `/images/avatars/upload/`, `/store/`, `config.php`)
    - [x] 8.2.5. Create `backup_info.txt` with metadata (timestamp, versions, file list)
    - [x] 8.2.6. Create `success.txt` marker file after all operations complete
    - [x] 8.2.7. Compress directory to `/backups/example-forum/YYYYMMDDHHMMSS-example-forum.tar.gz`
    - [x] 8.2.8. Remove uncompressed directory after successful compression
    - [x] 8.2.9. Add error handling and logging throughout the script

  - [x] 8.3. Make backup.sh executable in Docker container via Dockerfile

  - [x] 8.4. Test backup execution via Kamal: `kamal app exec --reuse '/scripts/backup.sh'`

  - [x] 8.5. Create `scripts/backup-cleanup.sh` for retention policy:

    - [x] 8.5.1. Keep 7 daily backups (delete older daily backups)
    - [x] 8.5.2. Keep 12 monthly backups (first backup of each month)
    - [x] 8.5.3. Keep all annual backups (first backup of each year)
    - [x] 8.5.4. Log cleanup operations

  - [x] 8.6. Create `scripts/restore.sh` for backup restoration:

    - [x] 8.6.1. Accept backup filename as parameter
    - [x] 8.6.2. Extract tar.gz to temporary directory
    - [x] 8.6.3. Verify `success.txt` exists
    - [x] 8.6.4. Restore database from `database.sql` with UTF-8 handling:
      - Use `mysql --default-character-set=utf8mb4`
      - Verify database charset is utf8mb4 before restore
      - Check collation compatibility (utf8mb4_unicode_ci)
    - [x] 8.6.5. Restore files to appropriate directories
    - [x] 8.6.6. Fix file permissions after restore
    - [x] 8.6.7. Clear phpBB cache after restore

  - [x] 8.7. Configure cron job on server to run backup script daily at 3:00 AM UTC

  - [x] 8.8. Create backup monitoring:

    - [x] 8.8.1. Script to check latest backup age
    - [x] 8.8.2. Script to verify backup integrity (check tar.gz is valid)
    - [x] 8.8.3. Add backup status to health check endpoint

  - [x] 8.9. Document backup/restore procedures in `example-forum-deploy/README.md`

  - [x] 8.10. Test complete backup and restore cycle on a test deployment
