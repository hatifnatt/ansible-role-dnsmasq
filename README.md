# dnsmasq

Set up [Dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html) on Debian-like systems.

## Requirements

None

## Role Variables

Check [defaults/main.yml](defaults/main.yml) and [vars/Debian.yml](/vars/Debian.yml) for available variables and it's default values.

Some complex variables will be explained below.

* `dnsmasq_conf` default `[]` data for main configuration file, if no data provided config file won't be updated

```yaml
---
# each config on it's own line
dnsmasq_conf:
  - "# main configuration file for dnsmasq, surrounding quotes are required"
  - port=53
  - "no-hosts"

# multiline mode - easy to write comments
---
dnsmasq_conf:
  - |
    # easy comment in multiline format
    port=53
    no-hosts

# to remove all configuration from file add single line with comment
# dnsmasq_conf: [] won't work, config file simply won't be updated
---
dnsmasq_conf:
  - "# empty file"
```

* `dnsmasq_dnsmasq_d_config_files` default `{}` create configuration files in `/etc/dnsmasq.d/`

```yaml
dnsmasq_dnsmasq_d_config_files:
  # By default it's mandatory to use .conf extension for configuration files,
  # files fith other extensions won't be loaded at startup
  00-defaul.conf:
    state: present
    config:
      - port=53
      - listen-address={{ ansible_lo['ipv4']['address'] }}
      - bind-interfaces
      - |
        # disable DHCP
        no-dhcp-interface=*
  01-absent.conf:
    state: absent
    config:
      - |
        # some lgeacy config
        address=/www.example.com/192.168.2.3
```

* `dnsmasq_dnsmasq_d_hosts_files_dir` default `/etc/dnsmasq.d/hosts` directory where host files will be created
* `dnsmasq_dnsmasq_d_hosts_files` default `{}` create `hosts` files from variable data

```yaml
dnsmasq_dnsmasq_d_hosts_files:
  # key name will be usead as a filename which will be saved in /etc/dnsmasq.d
  # in this example /etc/dnsmasq.d/hosts/hosts file will be created
  hosts:
    state: present
    records:
      - 192.168.5.99 dev.team.company.org
  # it's recommended to use .hosts extension to distinguish hosts files
  # from a main dnsmasq configuration files which usually have a .conf extension
  mycompany.tld.hosts:
    state: present
    records:
      - 192.168.2.10 sevice.mycompany.tld
  # if 'state' is set to 'absent' file will be deleted
  legacy.hosts:
    state: absent
    records:
      - "# comment - surrounding quotes are required"
      - 10.12.33.5 old.domain.tld
  # if no 'state' is set, default is 'present'
  # i.e. nostate.hosts file will be created although 'state' is not set explicitly
  nostate.hosts:
    records:
      - |
        # record with a comment, no quotes required in this form
        10.18.2.6 foo.mydomain.loc
      - |
        # multiple records in one list item
        10.18.2.4 myservice.mydomain.loc
        10.18.2.3 myservice.mydomain.loc
```

* `dnsmasq_dnsmasq_d_hosts_config_create` default `true`  
  `dnsmasq_dnsmasq_d_hosts_config_filename` default `/etc/dnsmasq.d/90-hosts.conf`  
  Create configuration file at location defined in `dnsmasq_dnsmasq_d_hosts_config_filename` variable.
  All hosts files defined in `dnsmasq_dnsmasq_d_hosts_files` with `state: present` will be referenced via `addn-hosts` parameter. For example above generated configuration file will look like

```
addn-hosts=/etc/dnsmasq.d/hosts/hosts
addn-hosts=/etc/dnsmasq.d/hosts/mycompany.tld.hosts
addn-hosts=/etc/dnsmasq.d/hosts/nostate.hosts
```

* `dnsmasq_default_config_dir` default `/etc/dnsmasq.d,*.conf` only load `*.conf` files from `/etc/dnsmasq.d` by default

## Dependencies

None

## Example Playbook

### Install dnsmasq on the target hosts

Install package, enable and start dnsmasq system service.

```yaml
- name: Install dnsmasq
  hosts: localhost
  roles:
    - hatifnatt.dnsmasq
```

### Local dns caching

```yaml
- name: Setup dnsmasq
  hosts: localhost
  pre_tasks:
    - name: Create resolv-file for dnsmasq
      ansible.builtin.copy:
        content: |
          nameserver 1.1.1.1
          nameserver 8.8.4.4
        dest: /etc/resolv.dnsmasq
  roles:
    - hatifnatt.dnsmasq
  vars:
    dnsmasq_conf:
      - |
        port=53
        listen-address={{ ansible_lo['ipv4']['address'] }}
        bind-interfaces
    dnsmasq_dnsmasq_d_files_present:
      10-cache.conf:
        state: present
        config:
          - |
            domain-needed
            bogus-priv
            no-hosts
            dns-forward-max=150
            cache-size=1000
            neg-ttl=180
            resolv-file=/etc/resolv.dnsmasq
            no-poll
```

## License

MIT

## Credits

Inspired by [Oefenweb ansible-dnsmasq](https://github.com/Oefenweb/ansible-dnsmasq/) role.
