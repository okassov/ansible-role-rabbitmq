# Ansible role for RabbitMQ

Ansible role to install and manage [RabbitMQ](https://www.rabbitmq.com/).

- Install and configure the Erlang
- Install and configure the RabbitMQ
- Configure RabbitMQ users
- Configure RabbitMQ authentication
- Configure RabbitMQ vhosts, queues
- Configure clustering (optional)

#### Variables

```yaml
rabbitmq_config: []
# - queue_name: test
#   durable: true
#   exchange_name: test
#   type: direct
#   routing_key: test
#   tags: "ha-mode=all,ha-sync-mode=automatic"
# - queue_name: test-quorum
#   durable: true
#   exchange_name: test-quorum
#   type: direct
#   routing_key: test
#   queue_type: quorum
#   tags: "ha-mode=all,ha-sync-mode=automatic"
# - policy_pattern: ".*"
#   vhost: apps
#   tags: "ha-mode=all,ha-sync-mode=automatic"

# Defines if rabbitmq ha should be configured
rabbitmq_config_ha: false

rabbitmq_config_service: false
rabbitmq_config_file: etc/rabbitmq/rabbitmq.config.j2
rabbitmq_config_env_file: etc/rabbitmq/rabbitmq-env.conf.j2
rabbitmq_env_config: {}

# current version if not defined
rabbitmq_debian_version_defined: true
rabbitmq_debian_version: 3.8.18-1

# Defines if setting up a rabbitmq cluster
rabbitmq_enable_clustering: false
# Defines the inventory host that should be considered master
rabbitmq_master: None

rabbitmq_erlang_cookie_file: /var/lib/rabbitmq/.erlang.cookie

rabbitmq_listen_port: 5672
rabbitmq_listeners: []
# - 127.0.0.1
# - '::1'

# Uncomment to set cluster partition handling strategy (https://www.rabbitmq.com/partitions.html)
#rabbitmq_cluster_partition_handling: ignore

rabbitmq_ssl_enable: false
rabbitmq_ssl_port: 5671
rabbitmq_ssl_listeners: []
# - 127.0.0.1
# - "::1"

rabitmq_ssl_options: {}
# cacertfile: '"/path/to/testca/cacert.pem"'
# certfile: '"/path/to/server/cert.pem"'
# keyfile: '"/path/to/server/key.pem"'
# verify: verify_peer
# fail_if_no_peer_cert: "false"

rabbitmq_redhat_repo_key: https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc
rabbitmq_redhat_package: "rabbitmq-server-{{ rabbitmq_redhat_version }}-1.el{{ ansible_distribution_major_version }}.noarch.rpm"
rabbitmq_redhat_url: "https://dl.bintray.com/rabbitmq/rpm/rabbitmq-server/v3.8.x/el/{{ ansible_distribution_major_version }}/noarch"
rabbitmq_redhat_version: 3.8.11

# Define extra vhosts to be created
rabbitmq_extra_vhosts: []
# - name: /
#   state: present

# Define admin user to create in order to login to WebUI
rabbitmq_users:
  - name: rabbitmqadmin
    password: rabbitmqadmin
    vhost: /
    configure_priv: ".*"
    read_priv: ".*"
    write_priv: ".*"
    # Define comma separated list of tags to assign to user:
    # management,policymaker,monitoring,administrator
    # required for management plugin.
    # https://www.rabbitmq.com/management.html
    tags: administrator

# comma separated list of plugins to enable
rabbitmq_plugins: "rabbitmq_management"
```

#### Usage

Example of inventory.yaml for Single RabbitMQ

```yaml
---
rabbitmq:
  vars:
    rabbitmq_master: "node1"
    rabbitmq_users:
      - name: admin
        password: admin
        vhost: /
        configure_priv: ".*"
        read_priv: ".*"
        write_priv: ".*"
        tags: administrator
    rabbitmq_plugins: "rabbitmq_management"
    rabbitmq_config:
      - queue_name: test
        durable: true
        exchange_name: test
        type: direct
        routing_key: test
  hosts:
    node1:
      ansible_host: 192.168.1.200
```

Example of inventory.yaml for Cluster RabbitMQ

```yaml
---

all:
  children:
    rabbitmq:

rabbitmq:
  vars:
    rabbitmq_master: "node1"
    rabbitmq_config_ha: true
    rabbitmq_enable_clustering: true
    rabbitmq_users:
      - name: admin
        password: admin
        vhost: /
        configure_priv: ".*"
        read_priv: ".*"
        write_priv: ".*"
        tags: administrator
    rabbitmq_plugins: "rabbitmq_management"
    rabbitmq_config:
      - queue_name: test
        durable: true
        exchange_name: test
        type: direct
        routing_key: test
  hosts:
    node1:
      ansible_host: 192.168.1.200
    node2:
      ansible_host: 192.168.1.201
    node3:
      ansible_host: 192.168.1.202
```

Example of playbook.yaml

```yaml
---
- hosts: rabbitmq
  roles:
    - rabbitmq
```
