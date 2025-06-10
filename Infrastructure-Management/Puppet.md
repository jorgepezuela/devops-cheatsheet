# 🤖 Puppet Cheat Sheet

![puppet](../Pictures/puppet.png)

## 📘 Introduction

[Puppet](https://puppet.com) is an **open-source configuration management tool** that automates infrastructure provisioning, configuration, and management. It uses a **declarative language** to describe the desired state of your systems.

Puppet supports both **agent-master** and **agentless (bolt)** architectures, making it powerful for large-scale environments.

---

## 🧠 Key Concepts

| Term         | Description                                                                |
| ------------ | -------------------------------------------------------------------------- |
| **Manifest** | A file written in Puppet DSL (.pp) that describes desired system state.    |
| **Module**   | A collection of manifests, templates, files, etc., organized in structure. |
| **Class**    | Reusable block of Puppet code.                                             |
| **Resource** | Basic unit that describes something (like a package or service).           |
| **Facts**    | System information gathered by **Facter**.                                 |
| **Catalog**  | Compiled version of the manifests specific to a node.                      |
| **Node**     | A client machine being managed.                                            |

---

## 🧾 Puppet Commands

<details>
<summary>🟢 Beginner Commands (Click to Expand)</summary>

### 🔹 Check Version

```bash
puppet --version
```

### 🔹 Apply Manifest Locally

```bash
puppet apply example.pp
```

### 🔹 Validate Syntax of Manifest

```bash
puppet parser validate example.pp
```

### 🔹 Format Manifests (Linting)

```bash
puppet parser validate example.pp
puppet-lint example.pp
```

### 🔹 List Available Facts

```bash
facter
facter os
```

### 🔹 View Help

```bash
puppet help
puppet help apply
```

</details>

---

<details>
<summary>🟡 Intermediate Commands (Click to Expand)</summary>

### 🔹 Puppet Resource (Inspect or Manage)

```bash
puppet resource <type>
puppet resource user root
puppet resource service ssh
```

### 🔹 Generate New Module Skeleton

```bash
puppet module generate yourname-modulename
```

### 🔹 Install a Module

```bash
puppet module install puppetlabs-apache
```

### 🔹 List Installed Modules

```bash
puppet module list
```

### 🔹 Check Current Puppet Config

```bash
puppet config print
puppet config print all
```

</details>

---

<details>
<summary>🔴 Advanced Commands (Click to Expand)</summary>

### 🔹 Agent Commands

```bash
puppet agent -t
puppet agent -t --debug
```

### 🔹 Manage Certificates

```bash
puppetserver ca list
puppetserver ca sign --certname node.example.com
puppetserver ca revoke --certname node.example.com
puppetserver ca clean --certname node.example.com
```

### 🔹 PuppetDB Query

```bash
puppet query 'inventory[certname] { facts.os.name = "Ubuntu" }'
```

### 🔹 Run Task with Bolt

```bash
bolt command run "uptime" --targets localhost
bolt plan run myplan
```

### 🔹 Testing & Debugging

```bash
puppet apply --noop file.pp
puppet apply --debug file.pp
puppet lookup varname
puppet describe <type>
```

### 🔹 System & Config

```bash
puppet config print <setting>
puppet facts show
puppet module search apache
puppet doc <module>
puppet resource --to_yaml
```

</details>

---

## 🟢 Beginner Level

### 🔹 Installing Puppet (Agent/Master)

```bash
# Install Puppet (Debian/Ubuntu)
sudo apt install puppet

# Check version
puppet --version
```

---

### 🔹 First Manifest Example

```puppet
# hello.pp
file { '/tmp/hello.txt':
  ensure  => present,
  content => "Hello from Puppet!",
}
```

Run it:

```bash
puppet apply hello.pp
```

---

### 🔹 Resource Types

| Type        | Example                             |
| ----------- | ----------------------------------- |
| **file**    | Manage files, directories, symlinks |
| **package** | Install, remove software            |
| **service** | Ensure a service is running/stopped |
| **user**    | Manage system users                 |

```puppet
# Install nginx and ensure it runs
package { 'nginx':
  ensure => installed,
}

service { 'nginx':
  ensure => running,
  enable => true,
}
```

---

### 🔹 Variables

```puppet
$greeting = "Hello, World"
notice($greeting)
```

---

### 🔹 Conditionals

```puppet
if $osfamily == 'Debian' {
  notice("Debian-based system")
} else {
  notice("Other OS")
}
```

---

## 🟡 Intermediate Level

### 🔸 Facts and Facter

View system facts:

```bash
facter
facter os
```

Use in manifests:

```puppet
if $facts['os']['family'] == 'RedHat' {
  package { 'httpd': ensure => installed }
}
```

---

### 🔸 Classes

```puppet
class apache {
  package { 'apache2': ensure => installed }
  service { 'apache2': ensure => running }
}
```

Include it:

```puppet
include apache
```

---

### 🔸 Modules

```bash
puppet module generate yourname-apache
puppet module install puppetlabs-apache
```

Structure:

```
apache/
├── manifests/
│   └── init.pp
├── files/
├── templates/
```

Use:

```puppet
class { 'apache': }
```

---

### 🔸 Templates (ERB)

File: `templates/vhost.erb`

```erb
<VirtualHost *:80>
  ServerName <%= @servername %>
</VirtualHost>
```

Manifest:

```puppet
file { '/etc/httpd/conf.d/vhost.conf':
  content => template('apache/vhost.erb'),
}
```

---

### 🔸 Puppet Apply vs Agent

| Mode      | Usage                                  |
| --------- | -------------------------------------- |
| **Apply** | Local apply of manifests               |
| **Agent** | Connects to master and applies catalog |

---

## 🔴 Advanced Level

### 🔹 Puppet Master-Agent Setup

* **Puppet Server**: Central server managing infrastructure.
* **Agent**: Node that pulls configuration from the server.

```bash
# On agent
puppet agent -t
```

Sign certs:

```bash
puppetserver ca list
puppetserver ca sign --certname <agent-fqdn>
```

---

### 🔹 Environments

Used to separate dev, staging, prod configs.

Directory structure:

```
/etc/puppetlabs/code/environments/
├── production/
│   └── manifests/
├── development/
```

---

### 🔹 Hiera (Hierarchical Data Lookup)

Configure external data in YAML:

```yaml
# hiera.yaml
version: 5
defaults:
  datadir: data
  data_hash: yaml_data

# data/common.yaml
apache::port: 80
```

Access in Puppet:

```puppet
$port = lookup('apache::port')
```

---

### 🔹 PuppetDB

Central storage for catalog, fact, and report data.

Query:

```puppet
query_nodes(['=', 'catalog_environment', 'production'])
```

---

### 🔹 Bolt (Agentless Task Runner)

```bash
bolt command run 'uptime' --targets localhost
bolt plan run myplan
```

Write plans in YAML or Puppet DSL.

---

## 📌 Useful Puppet CLI Commands

| Command                          | Description                   |
| -------------------------------- | ----------------------------- |
| `puppet apply <file.pp>`         | Apply a manifest locally      |
| `puppet agent -t`                | Trigger agent run             |
| `puppet resource <type> <name>`  | View current resource state   |
| `puppet module install <name>`   | Install a module              |
| `puppet config print all`        | Print all config settings     |
| `puppet parser validate file.pp` | Validate syntax of manifest   |
| `facter`                         | Show system facts             |
| `puppet doc <module>`            | Generate module documentation |

---

## 📚 Learning Resources

* 📘 [Official Docs](https://puppet.com/docs/puppet/latest/puppet_index.html)
* 📦 [Forge Modules](https://forge.puppet.com/)
* 🧪 [Bolt (Task Runner)](https://puppet.com/docs/bolt/latest/bolt.html)
* 📖 [Puppet DSL Cheat Sheet](https://puppet.com/docs/puppet/latest/lang_summary.html)
* 🧠 [Learn Puppet Free Courses](https://learn.puppet.com)

---