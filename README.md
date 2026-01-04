# SonarQube + PostgreSQL + pgAdmin + Traefik (HTTPS, IPv4/IPv6 Dual Stack)

This stack provides:

* **SonarQube (Community Edition)**
* **PostgreSQL 17** as SonarQube database
* **pgAdmin 4** to manage PostgreSQL
* **Traefik v2.11** reverse proxy with automatic HTTPS
* **Dual-stack Docker network (IPv4 + IPv6)**
* Persistent storage for all services

Everything runs securely behind Traefik with friendly hostnames:

| Service               | URL                                                    |
| --------------------- | ------------------------------------------------------ |
| **Traefik Dashboard** | [https://traefik.localhost](https://traefik.localhost) |
| **SonarQube**         | [https://sonar.localhost](https://sonar.localhost)     |
| **pgAdmin**           | [https://pgadmin.localhost](https://pgadmin.localhost) |

`.localhost` automatically resolves to `127.0.0.1`, so no hosts file changes are needed.

Open hosts file `sudo vim /etc/hosts`

Add the lines below:

```
127.0.0.1   sonar.local
127.0.0.1   pgadmin.local
127.0.0.1   traefik.local
```

## 🧱 Architecture

```
Browser  →  Traefik (HTTPS)
                     │
                     ├── sonar.localhost → SonarQube
                     ├── pgadmin.localhost → pgAdmin
                     └── traefik.localhost → Dashboard
                                      │
                               PostgreSQL
```

---

## 🚀 Running

### Start services

```bash
docker compose up -d
```

### Stop services

```bash
docker compose down
```

> ⚠️ Do **NOT** use `down -v` or you will erase all persisted data.

---

## 🔐 HTTPS Certificates

Traefik is configured with:

* HTTP → HTTPS redirect
* ACME TLS challenge
* Certificate storage persisted in volume `traefik_letsencrypt`

Certificates are stored in:

```
/letsencrypt/acme.json
```

persisted via:

```
traefik_letsencrypt
```

---

## 🧩 Services Explained

### 🧭 Traefik

* Acts as reverse proxy
* Terminates TLS
* Exposes dashboard
* Reads Docker labels dynamically
* Does **not** expose containers unless `traefik.enable=true`

Dashboard router:

```
Host(`traefik.localhost`)
service api@internal
```

---

### 🧪 SonarQube

Runs Community Edition with:

* Persistent data
* Logs
* Extensions
* Temporary files

Environment variables configure DB connection:

```
SONAR_JDBC_URL
SONAR_JDBC_USERNAME
SONAR_JDBC_PASSWORD
```

Accessible at:

👉 [https://sonar.localhost](https://sonar.localhost)

Database credentials are stored permanently in PostgreSQL volume, so **SonarQube admin password does NOT reset** when containers restart.

---

### 🗄 PostgreSQL

* Image: `postgres:17`
* Database: `sonar`
* User: `sonar`
* Password: `sonar`
* Data persists in volume `postgresql`

> ✅ Correct persistent folder:

```
/var/lib/postgresql/data
```

So DB survives restart/shutdown.

---

### 🧰 pgAdmin

Web interface for PostgreSQL.

Credentials:

```
Email: admin@local.com
Password: admin
```

Servers are auto-registered using mounted `servers.json`.

- At same path on your docker-compose file, create `pgadmin` folder then, create `servers.json` file as below

```
{
  "Servers": {
    "1": {
      "Name": "PostgreSQL - SonarQube",
      "Group": "Databases",
      "Host": "db",
      "Port": 5432,
      "MaintenanceDB": "sonar",
      "Username": "sonar",
      "SSLMode": "prefer"
    }
  }
}

```
Accessible at:

👉 [https://pgadmin.localhost](https://pgadmin.localhost)

Data persists in:

```
pgadmin_data
```

So login, preferences, and saved connections survive restarts.--

## 🌐 Networking

Two custom networks exist:

### `ipv4`

Standard IPv4 only bridge (not currently used)

### `dual`

Dual-stack network:

```
IPv4: 192.168.2.0/24
IPv6: 2001:db8:2::/64
```

Every container receives:

* Static internal IPv4 in this subnet
* Static IPv6 in the IPv6 subnet

This enables:

* Modern networking testing
* Compatibility with IPv6 environments
* Consistent internal addressing

---

## 💾 Persistent Volumes

| Volume               | Purpose                       |
| -------------------- | ----------------------------- |
| sonarqube_data       | Sonar application data        |
| sonarqube_extensions | Plugins                       |
| sonarqube_logs       | Logs                          |
| sonarqube_temp       | Temp runtime files            |
| postgresql           | PostgreSQL database data      |
| pgadmin_data         | pgAdmin state / saved servers |
| traefik_letsencrypt  | SSL certificates              |

Data is preserved unless explicitly deleted.

---

## 🔎 Useful Commands

### See assigned IPs

```bash
docker ps -q | xargs docker inspect \
  -f '{{.Name}} {{range .NetworkSettings.Networks}}{{.IPAddress}} {{.GlobalIPv6Address}}{{end}}'
```

### View logs

```bash
docker logs traefik
docker logs sonarqube
docker logs postgresql
docker logs pgadmin
```

---

## ✅ Summary

* Fully HTTPS-secured environment
* All services behind Traefik
* Persistent data
* PostgreSQL pre-configured
* pgAdmin auto-connected
* Dual-stack network enabled
* Ready for dev, test, lab, and learning environments

