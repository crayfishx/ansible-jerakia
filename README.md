# ansible-jerakia

## Description

This is an ansible lookup plugin for integrating with Jerakia

### Jerakia

[Jerakia](http://jerakia.io) is a hierarchical lookup tool. It supports pluggable datasources and the ability to override values in the hierarchy using information about the requestor. See [The official Jerakia site](http://jerakia.io) for more general information on Jerakia.

### Description

This plugin provides an interface between Ansible and Jerakia by way of a [lookup plugin](http://docs.ansible.com/ansible/latest/playbooks_lookups.html). Further plugin types are being investigated.

A hierarchical lookup is made by applying a _scope_ against a hierarchy of lookup paths. A scope is simply information about the requestor, in this case the Ansible node, that it used in determining what data to return. A scope could consist of anything, typical things would be the hostname, environment and location of the node. Using the scope provided in the lookup, Jerakia will run through a hierarchy and return the relevant value.

For the examples in the next section, we assume that Jerakia Server is installed and running on it's default port on the local machine and that we have the following default policy configured in Jerakia

```ruby
# /etc/jerakia/policy.d/default.rb

policy :default do
  lookup :default do
    datasource :file, {
      :format => :yaml,
      :docroot => "/var/lib/jerakia/data",
      :searchpath => [
        "node/#{scope[:hostname]}",
        "environment/#{scope[:environment]}",
        "operating_system/#{scope[:osfamily]}",
        "common"
      ]
    }
  end
end
```

See [the official documentation](http://jerakia.io/basics/lookups/) for more information about Jerakia policies and lookups.

### Configuration

The plugin will look for the `ANSIBLE_JERAKIA_CONFIG` environment variable to determine the location of the config file. If the variable is not set, it will look for a `jerakia.yaml` file in the directory where your playbook is located. The config file must contain an [authentication token](http://jerakia.io/server/tokens) to use to connect to Jerakia Server. We can also use the `scope` hash to map Ansible variables (facts) to the scope values used by the Jerakia policy above, eg:

```yaml
token: ansible:52cbf789c7837f9a4aef0d259c00d131f0f2a47894519c273c64a608de1382cba4221447752e9ac2
scope:
  hostname: ansible_nodename
  environment: ansible_local.custom.environment
  osfamily: ansible_os_family
```

Note that the scope keys in this example are the scope values used in the Jerakia policy, mapped to values which are Ansible variables (facts) for that request.

Supported configuration parameters of `jerakia.yaml` are:

| Parameter | Description | Required | Default |
|-----------|-------------|:--------:|:-------:|
| `token`   | The Jerakia token to use to authenticate against Jerakia server | Yes | - |
| `scope`   | A hash containing the scope to use for the request, the values will be resolved as Ansible facts. Use a dot notation to dig deeper into nested hash facts (see `environment` above) | No | - |
| `protocol` | The URL protocol to use | No | `http` |
| `host`    | Hostname of the Jerakia Server | No | `localhost` |
| `port`    | Jerakia port to connect to | No | `9843` |
| `version` | Jerakia API version to use | No | `1` |
| `policy`  | Jerakia policy to use for the lookups | No | `default` |

### Usage

Once configured with a jerakia.yaml, you can call the Jerakia lookup plugin directly from your playbooks using the `lookup` method. The first argument is always the name of the plugin, `jerakia`, and the following arguments contain the namespace and key to lookup, which is formatted as `<namespace>/<key>`

```yaml
- hosts: all
  tasks:
    - debug: msg=" {{ lookup('jerakia', 'apache/port') }}"
```

In the above example, if we assume the value of `ansible_nodename` is `foo.enviatics.com`, `environment` is `dev` and `ansible_os_family` is `RedHat` then with the policy we have declared, this will cause Jerakia to look up the key `port` in the namespace `apache`, it will follow the following hierarchy of files looking for the key `port` and return the first value it finds:

* `/var/lib/jerakia/data/node/foo.enviatics.com/apache.yaml`
* `/var/lib/jerakia/data/environment/dev/apache.yaml`
* `/var/lib/jerakia/data/operating_system/RedHat/apache.yaml`
* `/var/lib/jerakia/data/common/apache.yaml`

So we would be able to define a default value for the Apache port in `common/apache.yaml` but then have the ability to override this value based on a specific node, environment or operating system type. Note that the structure of the hierarchy here is purely an example, and is entirely configurable to suit your specific environment and needs.

### Contributing

This is a very new project so contributions are always welcome, please submit a pull request in the first instance.

### Maintainer

This project is maintained by Craig Dunn <craig@craigdunn.org> - @crayfishx

### License

This project is licensed under the Apache 2.0 license, please the the LICENSE file included with this software.
