Setting up an active-active PostgreSQL cluster on RHEL 8 using SymmetricDS entails a combination of PostgreSQL and SymmetricDS configurations. SymmetricDS is a software application that can be used to synchronize data across multiple relational databases, file systems, and cloud services. It supports PostgreSQL and can be used to achieve an active-active PostgreSQL cluster.

Here's a high-level guide to set up an active-active PostgreSQL cluster on RHEL 8 using SymmetricDS:

### Prerequisites:
1. Two RHEL 8 servers.
2. Proper networking between the servers.
3. Java Runtime Environment (preferably Java 8 or newer).

### Step-by-step guide:

#### 1. PostgreSQL Installation:

On both RHEL 8 nodes:

```bash
sudo dnf -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo dnf -qy module disable postgresql
sudo dnf -y install postgresql13 postgresql13-server
sudo /usr/pgsql-13/bin/postgresql-13-setup initdb
sudo systemctl enable --now postgresql-13
```

#### 2. Configure PostgreSQL:

On both nodes, make sure PostgreSQL listens on all IPs:

Edit `/var/lib/pgsql/13/data/postgresql.conf`:
```
listen_addresses = '*'
```

And update `/var/lib/pgsql/13/data/pg_hba.conf` to allow connections from the other node and SymmetricDS:

```
host    all             all             [other_node_ip]/32         trust
host    all             symmetricds     0.0.0.0/0                  md5
```

#### 3. Install Java:

On both nodes:
```bash
sudo dnf install java-11-openjdk
```

#### 4. Install SymmetricDS:

1. Download SymmetricDS from its official website.
2. Extract the downloaded file:

```bash
tar xzf symmetricds-x.x.x.tar.gz
cd symmetricds-x.x.x/bin
```

3. Run SymmetricDS:

```bash
./sym
```

#### 5. Configure SymmetricDS:

1. Create a `properties` file for each node. The contents will be similar, but the externalId and sync URL will differ:

For `node1`:

```
engine.name=pg1
db.driver=org.postgresql.Driver
db.url=jdbc:postgresql://localhost/symmetricds
db.user=symmetricds
db.password=password

registration.url=http://[node2_ip]:31415/sync/pg2
group.id=master
external.id=pg1
```

For `node2`:

```
engine.name=pg2
db.driver=org.postgresql.Driver
db.url=jdbc:postgresql://localhost/symmetricds
db.user=symmetricds
db.password=password

registration.url=http://[node1_ip]:31415/sync/pg1
group.id=master
external.id=pg2
```

2. Start SymmetricDS with the properties file:

```bash
./sym --engine [engine_name] &
```

Replace `[engine_name]` with either `pg1` or `pg2` depending on the node.

#### 6. Configure Tables for Synchronization:

Use the SymmetricDS tools or web interface to set up tables or entire schemas for synchronization between the two nodes.

#### 7. Test:

1. Add data on one node and ensure it gets replicated to the other node.
2. Also, ensure that data changes on the second node get replicated back to the first node, thus confirming the active-active setup.

This is a basic setup and doesn't account for additional security measures, optimizations, or other environment-specific configurations. Always ensure you read SymmetricDS documentation for more in-depth setup details and best practices. Also, test thoroughly before deploying in a production environment.
