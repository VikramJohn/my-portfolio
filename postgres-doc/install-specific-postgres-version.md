# Installing PostgreSQL 14.13 on Ubuntu 22.04 

You may need a specific version of postgres if your developers or software vendors recommend using one they have developed software for and done testing on. 

Here's how to install PostgreSQL 14.13 on an Ubuntu 22.04 server.

### Prerequisites

1. Remove existing postgres installations.

```bash
sudo apt remove postgresql
```

2. Ensure your system is up-to-date.

```bash
sudo apt update
sudo apt upgrade -y
```

### 1. Install dependencies

```
sudo apt install -y build-essential libreadline-dev zlib1g-dev libssl-dev libxml2-dev libxslt-dev libxml2-utils xsltproc libncurses5-dev libicu-dev libpam0g-dev libldap2-dev libkrb5-dev libperl-dev python3-dev tcl-dev flex bison wget pkg-config
```
See the [requirements for Postgresql 14](https://www.postgresql.org/docs/14/install-requirements.html) for an explanation of why these packages are required.

### 2. Install repository keys

```
sudo apt install curl ca-certificates
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc
```
[as stated here](https://wiki.postgresql.org/wiki/Apt)

### 3. Download the tarball and verify the download

```
cd /tmp
wget https://ftp.postgresql.org/pub/source/v14.13/postgresql-14.13.tar.gz
```

`ls -lh postgresql-14.13.tar.gz`

### 4. Extract the files

`tar -xzf postgresql-14.13.tar.gz`

### 5. Configure the build

`cd postgresql-14.13`

```
./configure --prefix=/usr/local/pgsql \
    --with-openssl \
    --with-libxml \
    --with-libxslt \
    --with-icu \
    --with-perl \
    --with-python \
    --with-tcl \
    --with-pam \
    --with-ldap
```
**Note:** The configure script checks for required libraries and will stop with an error if anything critical is missing. If you encounter an error, install the missing package with `sudo apt install [package]`.

### 6. Compile and test

```bash
make
```
Run regression tests to confirm postgres compiled correctly.

```bash
make check
```

Expected output:
```
===================
 All 212 tests passed.
===================
```
**Note**: To uninstall and remove built files, enter:

```
make uninstall
make clean
make distclean
```

### 7. Install PostgreSQL 14.13

```bash
sudo make install
```
Optionally, install offline postgres documentation.

```
sudo make install-docs
```

Install additional modules that add functionality like performance monitoring, encryption, and flexible data storage.

```bash
cd contrib
make
sudo make install
```

These extensions are installed but not enabled. Enable them using `CREATE EXTENSION` in your databases.

### 8. Create PostgreSQL system user

Return to the build directory.

```bash
cd /tmp/postgresql-14.13
```

Create the system user---**postgres**.
**Note**: The `-d` and `-s` flags set the home directory and the shell as bash.

```bash
sudo useradd -r -d /usr/local/pgsql -s /bin/bash postgres
```

### 9. Create the data directory

Create the PostgreSQL data directory and set **postgres** as the owner.

```bash
sudo mkdir -p /usr/local/pgsql/data
sudo chown -R postgres:postgres /usr/local/pgsql
```

### 10. Configure environment variables for postgres user

Set the `PATH` for the **postgres** user:

```bash
sudo -u postgres bash -c 'echo "export PATH=/usr/local/pgsql/bin:\$PATH" >> /usr/local/pgsql/.bashrc'
sudo -u postgres bash -c 'echo "export PGDATA=/usr/local/pgsql/data" >> /usr/local/pgsql/.bashrc'
```
**Note:** These variables are set only for the **postgres** user. If you want regular user accounts to use PostgreSQL commands directly, add the following to your `~/.bashrc`:

```bash
echo "export PATH=/usr/local/pgsql/bin:\$PATH" >> ~/.bashrc
source ~/.bashrc
```

Then you can run `psql`, `pg_dump`, etc. without switching to the **postgres** user.

### 11. Initialize database cluster

Switch to the **postgres** user and initialize the database.

```bash
sudo -u postgres /usr/local/pgsql/bin/initdb -D /usr/local/pgsql/data
```

The initialization creates the default database cluster and sets the locale and encoding.
You should see: "Success. You can now start the database server..."

### 12. Add the authentication method

Edit `pg_hba.conf` to set secure authentication methods. Use a text editor such as nano or Vi/Vim.

```bash
sudo -u postgres nano /usr/local/pgsql/data/pg_hba.conf
```
Scroll to the bottom of the file and change `trust` methods to `peer` and `scram-sha-256` (or `md5`) as shown below. 

```
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            scram-sha-256
# IPv6 local connections:
host    all             all             ::1/128                 scram-sha-256
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            scram-sha-256
host    replication     all             ::1/128                 scram-sha-256
```

### 13. Start the PostgreSQL server

Run the `start` command as **postgres** user.

```bash
sudo -u postgres /usr/local/pgsql/bin/pg_ctl -D /usr/local/pgsql/data -l /usr/local/pgsql/logfile start
```
You should see a "**server started**" message when the server is started.

Confirm that the server has started.

```bash
sudo -u postgres /usr/local/pgsql/bin/pg_ctl -D /usr/local/pgsql/data status
```

See the **Troubleshooting** section for possible issues and how to resolve them.

### 14. Create your application database and user

Enter PostgreSQL as the **postgres** user.

```bash
sudo -u postgres /usr/local/pgsql/bin/psql
```
You see the `postgres=#` prompt. 

  1. Create a user--**appuser** in the example below--who can create the roles and databases required for your application. Set a password for the user.

  ```sql
  CREATE ROLE appuser WITH SUPERUSER CREATEDB CREATEROLE LOGIN PASSWORD 'appuser';
  ```

  2. Create a database (`appdb` below).

  ```sql
  CREATE DATABASE appdb OWNER appuser;
  ```

  3. Check that the database was created.
  ```
  \l
  ```
  You should see `appdb` in the **List of databases** table with **appuser** as the owner.

  4. Exit postgres.
  ```
  \q
  ```
**Note:** In production environments, avoid giving application users `SUPERUSER` privileges. Grant only the specific permissions needed.

### 15. Configure PostgreSQL for network access

#### Allow non-local connections

Edit `postgresql.conf`. 

```bash
sudo -u postgres nano /usr/local/pgsql/data/postgresql.conf
```
The default setting accepts only connections from 'localhost'. Change this to '*' to allow other connections.

Find the line with `listen_addresses`, uncomment it and change `localhost` to `*`.

#### Set client authentication for the `appuser` user

Edit `pg_hba.conf`.

```bash
sudo -u postgres nano /usr/local/pgsql/data/pg_hba.conf
```

Append a line to the table at the bottom of the file.

```
host    appdb    appuser    YOUR_CLIENT_SUBNET    scram-sha-256
```
Replace `YOUR_CLIENT_SUBNET` with the relevant IP address and subnet size. For example, `192.168.1.0/24`.

#### Restart postgres

```bash
sudo systemctl restart postgresql
```

### 16. Configure Systemd Service for automatic startup 

Create a systemd service file for automatic startup:

```bash
sudo tee /etc/systemd/system/postgresql.service > /dev/null <<'EOF'
[Unit]
Description=PostgreSQL 14.13 Database Server
Documentation=https://www.postgresql.org/docs/14/
After=network.target

[Service]
Type=forking
User=postgres
Group=postgres
Environment=PGDATA=/usr/local/pgsql/data
ExecStart=/usr/local/pgsql/bin/pg_ctl start -D ${PGDATA} -l /usr/local/pgsql/logfile
ExecStop=/usr/local/pgsql/bin/pg_ctl stop -D ${PGDATA}
ExecReload=/usr/local/pgsql/bin/pg_ctl reload -D ${PGDATA}
TimeoutSec=300
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```

Enable and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable postgresql
sudo systemctl start postgresql
sudo systemctl status postgresql
```

---
### Verification

1. Check the installed version:

```bash
/usr/local/pgsql/bin/psql --version
```
You should see:
```bash
psql (PostgreSQL) 14.13
```

2. Test the connection as **appuser**.

```bash
/usr/local/pgsql/bin/psql -h 127.0.0.1 -U appuser -d appdb
```
Enter the password you set for **appuser** when prompted. You should see the PostgreSQL version again.

3. Try a test query.

```sql
SELECT version();
```
You should see the full PostgreSQL version string.

4. Create a test table.

```sql
CREATE TABLE test (id SERIAL PRIMARY KEY, data TEXT);
INSERT INTO test (data) VALUES ('PostgreSQL 14.13 is running');
SELECT * FROM test;
```
You should see:
| id |            data             
| --- | --- |
| 1 | PostgreSQL 14.13 is running |

5. Exit psql.
```sql
\q
```
---

### Troubleshooting Startup Issues

**Problem:** Address already in use

**Error:**
```
could not bind IPv4 address "127.0.0.1": Address already in use
HINT: Is another postmaster already running on port 5432?
```

**Solution:**

Check if PostgreSQL is already running:
```bash
sudo -u postgres /usr/local/pgsql/bin/pg_ctl status -D /usr/local/pgsql/data
```

If it shows "server is running," the server is already started - no action is needed.

If a different PostgreSQL installation is running:

1. Check for package-managed PostgreSQL

```bash
sudo systemctl status postgresql
```

2. If PostgreSQL is running, stop it.

```
sudo systemctl stop postgresql
sudo systemctl disable postgresql
```

Then try starting your installation again.

**Problem:** Permission denied

**Error:**
```
FATAL: could not open file "/usr/local/pgsql/data/postgresql.conf": Permission denied
```

**Solution:**

Fix file ownership:
```bash
sudo chown -R postgres:postgres /usr/local/pgsql
sudo -u postgres /usr/local/pgsql/bin/pg_ctl start -D /usr/local/pgsql/data
```

**Problem:** Lock file already exists

**Error:**
```
FATAL: lock file "postmaster.pid" already exists
```

**Solution:**

1. Check if the server is actually running.

```bash
ps aux | grep postgres
```

2. If no postgres processes are running, remove the stale lock file:
```bash
sudo -u postgres rm /usr/local/pgsql/data/postmaster.pid
sudo -u postgres /usr/local/pgsql/bin/pg_ctl start -D /usr/local/pgsql/data
```

3. If postgres processes are running (for example, a manually started instance conflicting with systemd), stop the server first.

```bash
sudo -u postgres /usr/local/pgsql/bin/pg_ctl stop -D /usr/local/pgsql/data
```
Then start via systemd.
```bash
sudo systemctl start postgresql
```

**Check Log File for Details**

If startup fails, check the log file for detailed error messages:
```bash
tail -50 /usr/local/pgsql/logfile
```