# Product Requirements Document: phpBB Forum Production Deployment

## Introduction/Overview

This PRD outlines the requirements for deploying the recovered phpBB forum (example-forum.com) to a production environment on Oracle Cloud Infrastructure using Kamal for deployment orchestration. The forum has been successfully recovered and upgraded to phpBB 3.3.15, and now needs to be deployed to serve its community of users with their historical data intact.

## Goals

1. Deploy the phpBB forum to Oracle Cloud VPS with zero data loss
2. Establish automated daily backup procedures to protect user data
3. Configure SSL/TLS encryption using Let's Encrypt for secure access
4. Set up a reproducible deployment process using Kamal
5. Ensure the forum is accessible at example-forum.com domain
6. Maintain the current Prosilver Dark theme and user experience

## User Stories

1. **As a forum administrator**, I want to deploy the forum to production so that users can access their community again.

2. **As a returning user**, I want to access my existing account and posts so that I can continue participating in discussions.

3. **As a system administrator**, I want automated daily backups so that data is protected against loss.

4. **As a developer**, I want to use Kamal for deployments so that updates can be pushed reliably and with zero downtime.

5. **As a visitor**, I want the forum to load over HTTPS so that my connection is secure.

## Functional Requirements

### 1. Infrastructure Setup

1.1. The system must run on Oracle Cloud VM.Standard.A1.Flex instance (2 OCPU, 6GB RAM, Ubuntu 24.04 ARM64)
1.2. The system must use the SSH user ubuntu@server01.example.com for access
1.3. The system must configure firewall to allow traffic on port 80 (HTTP)

### 2. Application Deployment

2.1. The system must use Kamal for Docker-based deployment orchestration
2.2. The system must run phpBB 3.3.15 with PHP 8.2 in a Docker container
2.3. The system must run MariaDB 10.1 as a Kamal accessory service
2.4. The system must preserve all existing forum data (73 users, 17,252 posts)
2.5. The system must include the Prosilver Dark theme with orange color scheme
2.6. The system must leverage Kamal's built-in rollback capabilities for failed deployments
2.7. The system must limit container memory usage to 2GB maximum

### 3. Domain and SSL Configuration

3.1. The system must be accessible via example-forum.com domain
3.2. The system must obtain SSL certificates from Let's Encrypt
3.3. The system must automatically renew SSL certificates before expiration
3.4. The system must initially use /etc/hosts for testing before DNS changes

### 4. Data Management

4.1. The system must import the existing database dump during initial deployment
4.2. The system must preserve uploaded files and avatars from the backup
4.3. The system must perform automated daily database backups
4.4. The system must store backups with the following retention policy: - 7 daily backups - 12 monthly backups - Infinite annual backups

### 5. Repository Structure

5.1. The deployment must use a separate Git repository named "example-forum-deploy"
5.2. The repository must include all necessary phpBB files
5.3. The repository must include Kamal configuration files
5.4. The repository must include backup scripts and configurations

### 6. Security Requirements

6.1. The system must create new admin credentials (not preserve old ones)
6.2. The system must properly secure the config.php file
6.3. The system must set appropriate file permissions for phpBB directories
6.4. The system must not expose sensitive configuration in Git

### 7. Email Configuration

7.1. The system must configure email delivery for password resets and notifications
7.2. The system must use Oracle Cloud Infrastructure Email Delivery service
7.3. The system must configure proper SPF/DKIM records for email deliverability

## Non-Goals (Out of Scope)

1. Multi-server deployment or horizontal scaling
2. Staging environment setup
3. CI/CD pipeline integration (deployments will be manual)
4. Advanced monitoring or alerting systems
5. Caching layers (Redis/Memcached)
6. Database clustering or replication
7. Custom phpBB extensions or modifications
8. Automated testing framework
9. Rate limiting or DDoS protection (handled by Cloudflare)
10. Oracle Cloud monitoring integrations
11. Automatic OS security updates

## Design Considerations

### Directory Structure

The deployment repository should follow this structure:

```
example-forum-deploy/
├── .kamal/
│   ├── hooks/
│   └── secrets
├── config/
│   └── deploy.yml
├── phpbb/
│   ├── files/
│   ├── images/
│   ├── store/
│   └── styles/
├── database/
│   └── initial_dump.sql
├── scripts/
│   └── backup.sh
├── Dockerfile
├── .env.example
├── .gitignore
└── README.md
```

### Backup Strategy

- Daily automated backups at 3:00 AM UTC
- Retention policy:
  - 7 daily backups
  - 12 monthly backups (taken from the daily backups on the 1st of each month)
  - Infinite annual backups (taken from the monthly backups on January 1st)
- Store backups in `/backup/example-forum/` on the VPS
- Include both database dumps and uploaded files

### phpBB Update Process

Recommended process for future phpBB version updates:

1. **Test in local environment**: Clone production data and test upgrade locally
2. **Create full backup**: Database dump + all files before any changes
3. **Schedule maintenance window**: Inform users of planned downtime
4. **Use Kamal for deployment**:
   - Build new Docker image with updated phpBB version
   - Deploy using `kamal deploy` for zero-downtime upgrade
   - If issues arise, use `kamal rollback` to revert
5. **Post-update verification**: Test all critical forum functions
6. **Keep detailed changelog**: Document all changes and issues encountered

## Technical Considerations

1. **ARM64 Architecture**: Ensure all Docker images are compatible with ARM64
2. **Memory Constraints**:
   - Total system RAM: 6GB
   - Container memory limit: 2GB maximum
   - Configure PHP memory_limit: 256MB
   - Configure MySQL innodb_buffer_pool_size: 512MB
3. **Storage**: Monitor disk usage for backups and uploaded files
4. **Network**: Configure Kamal to work with Oracle Cloud's networking
5. **Secrets Management**: Use Kamal's secret management for database passwords
6. **Volume Mounts**: Properly configure persistent volumes for database and uploads
7. **Email Delivery**: Configure Oracle Email Delivery service with proper authentication

## Success Metrics

1. Forum accessible at https://example-forum.com with valid SSL certificate
2. All 73 users can log in with password reset functionality
3. All 17,252 posts are accessible and properly formatted
4. Daily backups completing successfully without errors
5. Zero-downtime deployments achievable using Kamal
6. Page load times under 2 seconds for typical pages
7. 99.9% uptime over first month of operation
8. Email delivery working for password resets and notifications

## Resolved Decisions

Based on stakeholder input, the following decisions have been made:

1. **Rollback Strategy**: Use Kamal's built-in rollback functionality
2. **Backup Retention**: 7 daily, 12 monthly, infinite annual backups
3. **Email Service**: Oracle Cloud Infrastructure Email Delivery
4. **DDoS Protection**: Handled by Cloudflare (not in scope)
5. **Monitoring**: No Oracle Cloud specific integrations needed
6. **OS Updates**: Manual process only, no automatic updates
