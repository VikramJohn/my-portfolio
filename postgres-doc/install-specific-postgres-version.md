# Installing PostgreSQL 15.3 on Ubuntu 22.04

You may need a specific version of postgres if your developers or software vendors recommend using a specific version that they have developed software for and done testing on. 

Here's how to install PostgreSQL 15.3 on an Ubuntu 22.04 server.

### 1. Remove existing postgres
`sudo apt remove postgresql`

### 2. Ensure your system is up-to-date.
```
sudo apt update
sudo apt upgrade -y
```
### 3. Install dependencies
```
sudo apt install -y build-essential libreadline-dev zlib1g-dev libssl-dev libxml2-dev libxslt-dev libxml2-utils xsltproc libncurses5-dev libicu-dev libpam0g-dev libldap2-dev libkrb5-dev libperl-dev python3-dev tcl-dev flex bison wget
```
See the [requirements for Postgresql 15](https://www.postgresql.org/docs/15/install-requirements.html) for an explanation of why these packages are required.

### 4. 
