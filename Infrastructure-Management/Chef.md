# 🧑‍🍳 Chef Cheat Sheet

![chef](../Pictures/chef.png)

## 📘 Introduction

Chef is a configuration management tool written in Ruby and Erlang. It automates the process of configuring, deploying, and managing servers and infrastructure.

---

## 🟢 Beginner Level

### 🔹 Key Concepts

* **Node**: A machine (physical/virtual) managed by Chef.
* **Cookbook**: A collection of recipes and other config files.
* **Recipe**: The fundamental unit for configuration — contains Ruby code to define resources.
* **Resource**: A statement of configuration like `package`, `service`, `file`, etc.
* **Runlist**: A list of recipes/roles applied to a node.

---

## 🍳 Chef Commands

<details>
<summary>🟢 Beginner Commands (Click to Expand)</summary>

### 🔹 Check Version

```bash
chef --version
```

### 🔹 Generate Cookbook

```bash
chef generate cookbook my_cookbook
```

### 🔹 Generate Recipe

```bash
chef generate recipe my_cookbook default
```

### 🔹 Run Chef Client (Local Mode)

```bash
chef-client --local-mode --runlist 'recipe[my_cookbook]'
```

### 🔹 Validate Ruby Syntax

```bash
ruby -c recipes/default.rb
```

</details>

---

<details>
<summary>🟡 Intermediate Commands (Click to Expand)</summary>

### 🔹 Knife Bootstrap Node

```bash
knife bootstrap <IP_ADDRESS> -x user -P password --node-name node1
```

### 🔹 Upload Cookbook to Server

```bash
knife cookbook upload my_cookbook
```

### 🔹 Show Node Information

```bash
knife node show node1
```

### 🔹 Run Chef Client on Remote Node

```bash
knife ssh 'name:node1' 'sudo chef-client' -x user
```

</details>

---

<details>
<summary>🔴 Advanced Commands (Click to Expand)</summary>

### 🔹 Search Nodes or Data Bags

```bash
knife search node 'role:web'
knife data bag show users
```

### 🔹 Edit Node or Role

```bash
knife node edit node1
knife role edit webserver
```

### 🔹 Generate Role or Environment

```bash
chef generate role webserver
chef generate environment dev
```

### 🔹 Upload Roles/Environments

```bash
knife role from file roles/webserver.rb
knife environment from file environments/dev.rb
```

### 🔹 Test Cookbook (ChefSpec, InSpec)

```bash
chef exec rspec
chef exec inspec exec test/integration/default
```

</details>

---

### 🔹 Chef Setup

```bash
# Install Chef Workstation
curl -LO https://packages.chef.io/files/stable/chef-workstation/latest/el/7/chef-workstation-*.rpm
sudo rpm -Uvh chef-workstation-*.rpm
```

```bash
# Verify installation
chef -v
```

---

### 🔹 Create a Cookbook

```bash
chef generate cookbook my_cookbook
cd my_cookbook
```

---

### 🔹 Basic Recipe

```ruby
# recipes/default.rb
package 'nginx'

service 'nginx' do
  action [:enable, :start]
end
```

```bash
# Run recipe on local machine (Test Kitchen or chef-run)
chef-run 'localhost' my_cookbook
```

---

### 🔹 Common Resources

| Resource  | Example                                        |
| --------- | ---------------------------------------------- |
| `package` | `package 'nginx'`                              |
| `service` | `service 'nginx' { action [:start, :enable] }` |
| `file`    | `file '/etc/motd' { content 'Hello Chef' }`    |
| `user`    | `user 'deploy' { shell '/bin/bash' }`          |

---

## 🟡 Intermediate Level

### 🔸 Attributes

```ruby
# attributes/default.rb
default['my_cookbook']['greeting'] = 'Welcome to Chef!'

# Use in recipe
file '/etc/motd' do
  content node['my_cookbook']['greeting']
end
```

---

### 🔸 Templates

Templates are ERB files used to manage config files.

```bash
# generate template
mkdir templates/default
touch templates/default/index.html.erb
```

```erb
<!-- templates/default/index.html.erb -->
<h1>Hello <%= node['hostname'] %>!</h1>
```

```ruby
# recipes/default.rb
template '/var/www/html/index.html' do
  source 'index.html.erb'
end
```

---

### 🔸 Data Bags

```bash
# Create data bag and item
knife data bag create users
knife data bag from file users user1.json
```

```json
// users/user1.json
{
  "id": "user1",
  "uid": "1001",
  "shell": "/bin/bash"
}
```

```ruby
# Use in recipe
user_data = data_bag_item('users', 'user1')

user user_data['id'] do
  uid user_data['uid']
  shell user_data['shell']
end
```

---

### 🔸 Roles

```bash
# Create role file
knife role create webserver
```

```json
{
  "name": "webserver",
  "run_list": [
    "recipe[my_cookbook::default]"
  ]
}
```

---

### 🔸 Environments

Used to manage differences between dev, test, prod.

```bash
# Create environment
knife environment create dev
```

```json
{
  "name": "dev",
  "default_attributes": {
    "my_cookbook": {
      "greeting": "Welcome to Dev Environment"
    }
  }
}
```

---

## 🔴 Advanced Level

### 🔹 Custom Resources

```bash
# inside cookbooks/my_cookbook/resources/hello.rb
resource_name :hello

property :name, String, name_property: true

action :create do
  file "/tmp/#{name}" do
    content "Hello, #{name}!"
  end
end
```

```ruby
# recipes/default.rb
hello 'chef_user'
```

---

### 🔹 ChefSpec (Unit Testing)

```ruby
# spec/unit/recipes/default_spec.rb
require 'chefspec'

describe 'my_cookbook::default' do
  let(:chef_run) { ChefSpec::SoloRunner.new.converge(described_recipe) }

  it 'installs nginx' do
    expect(chef_run).to install_package('nginx')
  end
end
```

Run tests:

```bash
rspec
```

---

### 🔹 Test Kitchen (Integration Testing)

```bash
# .kitchen.yml
driver:
  name: vagrant

provisioner:
  name: chef_zero

platforms:
  - name: ubuntu-20.04

suites:
  - name: default
    run_list:
      - recipe[my_cookbook::default]
```

```bash
kitchen converge
kitchen verify
```

---

### 🔹 Policyfiles

Alternative to Berkshelf and runlists.

```bash
chef generate policyfile my_policy
```

```ruby
# Policyfile.rb
name 'my_policy'
run_list 'my_cookbook::default'
default_source :supermarket
cookbook 'my_cookbook', path: '.'
```

```bash
chef install
chef push my_org my_policy.lock.json
```

---

### 🔹 Chef Automate

Chef Automate provides UI and compliance, visibility, and workflow capabilities.

* Setup dashboards
* Integrate with InSpec for audits
* Workflow pipelines for cookbook CI/CD

---

### 🔹 Knife Tips

```bash
knife bootstrap IP_ADDRESS -x user -P password --node-name NODE_NAME
knife node list
knife cookbook upload my_cookbook
knife role from file webserver.json
```

---

## 📌 Best Practices

* Keep cookbooks modular and reusable.
* Use Berkshelf or Policyfiles to manage dependencies.
* Write tests (ChefSpec/Test Kitchen) for stability.
* Avoid hardcoding; use attributes or data bags.
* Prefer custom resources over LWRPs.