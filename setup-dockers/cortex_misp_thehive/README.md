# Cortex, MISP & TheHive Docker Deployment

## Overview

This folder contains a **Docker Compose** configuration that orchestrates a complete Security Operations Center (SOC) stack with:

- **TheHive 5.2** - Scalable incident response & case management platform
- **Cortex** - Automated malware analysis & threat intelligence engine
- **MISP** - Threat intelligence sharing platform
- **Cassandra** - NoSQL database for TheHive data persistence
- **Elasticsearch** - Search and analytics for alert data
- **MinIO** - S3-compatible object storage for TheHive
- **MySQL** - Database backend for MISP

## System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                   Docker Network: SOC_NET                   │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │   TheHive    │    │   Cortex     │    │     MISP     │  │
│  │   5.2        │    │   (Latest)   │    │ (Latest)     │  │
│  │  Port 9000   │    │  Port 9001   │    │ Port 8081    │  │
│  └──────┬───────┘    └──────┬───────┘    └──────┬───────┘  │
│         │                    │                    │           │
│         ▼                    ▼                    ▼           │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │  Cassandra   │    │Elasticsearch │    │ MISP MySQL   │  │
│  │  Port 9042   │    │  Port 9201   │    │              │  │
│  └──────────────┘    └──────────────┘    └──────────────┘  │
│         │                                       │             │
│         └───────────────────┬───────────────────┘             │
│                             │                                 │
│                      ┌──────▼──────┐                          │
│                      │    MinIO     │                          │
│                      │ Port 9002    │                          │
│                      └──────────────┘                          │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

## Files

### `docker-compose.yml`
Complete Docker Compose orchestration file defining all services and their configurations.

**Services Overview:**

#### 1. **TheHive (Port 9000)**
- **Image:** `strangebee/thehive:5.2`
- **Dependencies:** Cassandra, Elasticsearch, MinIO, Cortex
- **Memory Limit:** 1500 MB
- **Function:** Main incident response and case management platform
- **JVM Options:** `-Xms1024M -Xmx1024M` (1GB heap)
- **Storage:** Cassandra for data, MinIO for files

#### 2. **Cassandra (Port 9042)**
- **Image:** `cassandra:4`
- **Memory Limit:** 2048 MB
- **Function:** Distributed NoSQL database for TheHive persistence
- **Cluster Name:** TheHive
- **Heap Settings:** 1024MB max, 256MB new size
- **Volume:** `cassandradata` - Persistent database storage

#### 3. **Elasticsearch (Port 9201)**
- **Image:** `docker.elastic.co/elasticsearch/elasticsearch:7.17.9`
- **Memory Limit:** 2048 MB
- **Function:** Search and indexing engine for alerts
- **Cluster Name:** hive
- **Security:** Disabled (suitable for lab environments)
- **JVM Heap:** 1GB allocated
- **Note:** Port changed from 9200 to 9201 to avoid conflicts with Wazuh
- **Volume:** `elasticsearchdata` - Index persistence

#### 4. **MinIO (Port 9002)**
- **Image:** `quay.io/minio/minio`
- **Function:** S3-compatible object storage for file attachments
- **Credentials:** 
  - Username: `minioadmin`
  - Password: `minioadmin`
- **Volume:** `miniodata` - Persistent storage

#### 5. **Cortex (Port 9001)**
- **Image:** `thehiveproject/cortex:latest`
- **Function:** Automated analysis engine for suspicious files/URLs
- **JVM Options:** `-Xms1g -Xmx1g` (1GB heap)
- **Job Directory:** `/tmp/cortex-jobs`
- **Docker Socket:** Mounted for Docker-based analyzers
- **Volumes:**
  - Docker socket for container execution
  - Job directory for analysis tasks
  - Logs directory
  - Custom application configuration

#### 6. **MISP (Port 8081 HTTP, 8444 HTTPS)**
- **Image:** `coolacid/misp-docker:core-latest`
- **Function:** Threat intelligence sharing platform
- **Dependencies:** MISP MySQL database
- **Ports:**
  - HTTP: `8081` (mapped from container port 80)
  - HTTPS: `8444` (mapped from container port 443)

#### 7. **MISP MySQL**
- **Function:** Database backend for MISP
- **Managed by:** MISP container depends_on declaration

## Prerequisites

### System Requirements
- **CPU:** 4+ cores (8+ recommended for production)
- **RAM:** 8 GB minimum (16 GB recommended)
- **Disk Space:** 50 GB+ for all volumes and data
- **Docker:** Version 20.10+
- **Docker Compose:** Version 1.29+

### Required Software
```bash
# Install Docker
curl -fsSL https://get.docker.com | sh

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.x.x/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

## Deployment Instructions

### Step 1: Clone/Download Files
```bash
cd /path/to/setup-dockers/cortex_misp_thehive
```

### Step 2: Start Services
```bash
# Start all services in background
docker-compose up -d

# Check service health
docker-compose ps
```

### Step 5: Wait for Initialization
Initial startup takes 2-5 minutes as services initialize databases:
```bash
# Watch logs for TheHive startup completion
docker-compose logs -f thehive | grep -i "started"
```

### Step 6: Access Services

| Service | URL | Default Credentials |
|---------|-----|-------------------|
| **TheHive** | http://localhost:9000 | Default user setup on first login |
| **Cortex** | http://localhost:9001 | Requires admin key configuration |
| **MISP** | http://localhost:8081 | Check MISP documentation |
| **MinIO** | http://localhost:9002 | minioadmin / minioadmin |

## Configuration Details

### Network Configuration
- **Network Name:** `SOC_NET`
- **Type:** Bridge network
- **Service DNS:** Services communicate via container names (e.g., `thehive`, `cassandra`)

### Volume Mapping
| Volume | Mount Point | Purpose |
|--------|------------|---------|
| `cassandradata` | `/var/lib/cassandra` | Cassandra data persistence |
| `elasticsearchdata` | `/usr/share/elasticsearch/data` | Elasticsearch indices |
| `miniodata` | `/data` | S3 object storage |
| `thehivedata` | `/etc/thehive/application.conf` | TheHive config |
| `./cortex/logs` | `/var/log/cortex` | Cortex application logs |

### Memory & Resource Limits
- **TheHive:** 1500 MB RAM
- **Cassandra:** 2048 MB RAM
- **Elasticsearch:** 2048 MB RAM
- **Cortex:** No hard limit (1GB JVM heap)
- **MinIO:** No hard limit
- **MISP:** No hard limit

## Operational Commands

### View Service Status
```bash
docker-compose ps
docker-compose logs <service_name>
docker-compose logs -f --tail=50
```

### Stop Services
```bash
# Graceful shutdown
docker-compose down

# Stop without removing volumes (data preserved)
docker-compose stop
```

### Restart Services
```bash
# Restart all services
docker-compose restart

# Restart specific service
docker-compose restart thehive
```

### Access Service Shell
```bash
# Open shell in TheHive container
docker-compose exec thehive /bin/bash

# Access Cassandra CQL shell
docker-compose exec cassandra cqlsh

# Access Elasticsearch
docker-compose exec elasticsearch curl localhost:9200/_cat/health
```

### View Resource Usage
```bash
docker stats

# Specific service
docker stats thehive cassandra elasticsearch
```

## Troubleshooting

### Services Not Starting
**Symptom:** Containers exit immediately
```bash
# Check logs
docker-compose logs <service_name>

# Common causes:
# 1. Port already in use
# 2. Insufficient disk space
# 3. Insufficient RAM
```

**Solution:**
```bash
# Check port availability
lsof -i :9000  # TheHive
lsof -i :9001  # Cortex
lsof -i :9042  # Cassandra

# Free up disk space
df -h
```

### Cassandra Not Initializing
**Symptom:** TheHive fails with "Cannot reach Cassandra"
```bash
# Wait longer (Cassandra can take 3-5 minutes)
docker-compose logs cassandra | grep -i "ready"
```

### Elasticsearch Memory Issues
**Symptom:** Container killed due to out-of-memory
```bash
# Increase system swap
sudo sysctl -w vm.swappiness=60

# Or reduce Elasticsearch heap in docker-compose.yml
# Change: "ES_JAVA_OPTS=-Xms1g -Xmx1g"
# To: "ES_JAVA_OPTS=-Xms512m -Xmx512m"
```

### High Disk Usage
**Symptom:** Storage fills up quickly
```bash
# Clean up old containers and images
docker system prune -a

# Check volume sizes
du -sh cassandradata/ elasticsearchdata/ miniodata/

# Implement retention policies in services
```

## Production Deployment Notes

### Security Hardening
1. **Change Default Credentials:**
   - MinIO admin password (change `minioadmin`)
   - MISP admin user password

2. **Enable Network Security:**
   - Use `docker network inspect SOC_NET` to verify isolation
   - Consider using firewall rules for external access

3. **Enable Elasticsearch Security:**
   - Set `xpack.security.enabled=true` (requires license)
   - Or use a reverse proxy with authentication

4. **Backup Strategy:**
   - Regular backups of volumes
   - Cassandra backup procedures
   - Test restore procedures

### Performance Optimization
1. **Scale Resource Limits:**
   - Increase heap sizes for production load
   - Monitor JVM metrics: `docker stats`

2. **Database Tuning:**
   - Cassandra compaction settings
   - Elasticsearch index lifecycle management

3. **Load Balancing:**
   - Consider deploying multiple instances
   - Use reverse proxy (nginx) for load distribution

## Integration with Wazuh

Connect Wazuh alerts to TheHive:
1. Configure Wazuh integration (see `../Code/README.md`)
2. TheHive will receive alerts on port 9000
3. Cortex analyzers process suspicious artifacts
4. MISP provides threat intelligence context

## Cleanup & Reset

### Reset All Services (WARNING: Deletes all data)
```bash
docker-compose down -v

# Remove all volumes
docker volume rm cortex_misp_thehive_cassandradata \
                  cortex_misp_thehive_elasticsearchdata \
                  cortex_misp_thehive_miniodata \
                  cortex_misp_thehive_thehivedata
```

### Partial Reset
```bash
# Reset only Cassandra (keeps other services)
docker-compose down cassandra
docker volume rm cortex_misp_thehive_cassandradata
docker-compose up -d cassandra
```

## Documentation & Support

- **TheHive Documentation:** https://docs.thehive-project.org/
- **Cortex Documentation:** https://docs.thehive-project.org/cortex/
- **MISP Documentation:** https://misp.github.io/MISP/
- **Docker Compose Docs:** https://docs.docker.com/compose/
