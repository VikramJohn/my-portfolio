# [Work in Progress] Installing PostgreSQL 14.13 on Ubuntu 22.04 

You may need a specific version of postgres if your developers or software vendors recommend using a specific version that they have developed software for and done testing on. 

Here's how to install PostgreSQL 15.3 on an Ubuntu 22.04 server.

### 1. Remove existing postgres
`sudo apt remove postgresql`

### 2. Ensure your system is up-to-date.
```
sudo apt update
sudo apt upgrade -y
```
-----------Prerequisites? -------------------
### 3. Install dependencies
```
sudo apt install -y build-essential libreadline-dev zlib1g-dev libssl-dev libxml2-dev libxslt-dev libxml2-utils xsltproc libncurses5-dev libicu-dev libpam0g-dev libldap2-dev libkrb5-dev libperl-dev python3-dev tcl-dev flex bison wget pkg-config
```
See the [requirements for Postgresql 15](https://www.postgresql.org/docs/15/install-requirements.html) for an explanation of why these packages are required.

### 4. Install repository keys

```
sudo apt install curl ca-certificates
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc
```
[as stated here](https://wiki.postgresql.org/wiki/Apt)

### 5. Download the tarball and verify the download

```
cd /tmp
wget https://ftp.postgresql.org/pub/source/v14.13/postgresql-14.13.tar.gz
```

`ls -lh postgresql-14.13.tar.gz`

### 6. Extract the files

`tar -xzf postgresql-14.13.tar.gz`

### 7. Configure the build

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

### 8. Compile and test

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

### 9. Install PostgreSQL 14.13

```bash
sudo make install
```
Optionally, install offline postgres documentation.

```
sudo make install-docs
```

Install additional modules that add functionality like performance monitoring, encryption, and flexible data storage:

```bash
cd contrib
make
sudo make install
```

These extensions are installed but not enabled. Enable them using `CREATE EXTENSION` in your databases.

### 10. Create PostgreSQL system user

Return to the build directory.

```bash
cd /tmp/postgresql-14.13
```

Create the system user---postgres.
**Note**: The `-d` and `-s` flags set the home directory and the shell as bash.

```bash
sudo useradd -r -d /usr/local/pgsql -s /bin/bash postgres
```

### 11. Create the data directory

Create and set ownership of the PostgreSQL data directory:

```bash
sudo mkdir -p /usr/local/pgsql/data
sudo chown -R postgres:postgres /usr/local/pgsql
```
