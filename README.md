# Ansible role: Bind

Install and configure bind with dnssec

## Requirements

* Ansible >= 2.7
* bind version >= 9.8
* Debian Stretch and Buster

## Role variables

* `bind_options` - hash general bind options
* `bind_zones` - the dns zones
* `bind_dnssec` - hash with dnssec configuration
* `bind_tsig` - the tsig keys
* `bind_zones_subset` array to use in `extra-vars` with the list zones to update
* `bind_listen_ipv4` - enable or disable ip v4 support (default: true)
* `bind_listen_ipv6` - enable or disable ip v6 support (default: true)
## How to use

* `group_vars/dns-server/bind`

```yaml
bind_options:
  server-id: '"1"'

bind_zones:
  test.local:
    ns_primary: ns1.test.local
    mail: root@test.local
    serial: 2017092202
    options:
      auto-dnssec: maintain
      inline-signing: yes
    records:
      - { name: '@', type: ns, value: localhost. }
      - { name: hello, type: a, ttl: 5m, value: 1.2.3.4 }
      - { name: hello, type: caa, flag: 0, tag: issue, value: letsencrypt.org }
      - { name: hello, type: srv, priority: 0, weight: 5, port: 80, value: www }
  hello.local:
    ns_primary: ns1.hello.local
    mail: root@hello.local
    serial: 2017092201
    dnssec: no
    state: disabled
    records:
      - { name: '@', type: ns, value: localhost. }
      - { name: hello, type: a, value: 4.3.2.1 }
```

* `group_vars/dns-server/dnssec`

```yaml
bind_dnssec:
  test.local: 
    ksk:
      algorithm: 8
      digest: 3
      tag: 63805
      public_key: AwEAAbA3M8p+Cpf4k6mZKK8mb1eSIF8yDWXnpmI+i/Jm6CtIYMSigZ4B bmnN+r/SdpeeaPCP5RRZDO/6U0xs2zwPeLs=
      private_key: !vault |
        $ANSIBLE_VAULT;1.1;AES256
        33373964393565343638363964366133663235653931386664343435326362333031323130363362
        [...]
        65616337363634636365386166643133373331336333376430353663303563346236316532336532
        62376530646231346237
    zsk:
      algorithm: 8
      digest: 3
      tag: 11346
      public_key: AwEAAd9SkkrJQl4tOsK3zgtfZwmSJBzxU/NjApDZiKo6AVYVhDun6IIl Q/axOe901o+x/iUVwIs7cOMA5Z/h/8G8bq8=
      private_key: !vault |
        $ANSIBLE_VAULT;1.1;AES256
        37323036613735396364323363323464393731626466616262613033656264343765306238353934
        [...]
        38653039306430393564346636323966373265343032623430353765646639366536663566653836
        32643931393165643236

```

* `group_vars/dns-server/tsig`

```yaml
bind_tsig:
  all:
    - name: ddns_certbot_global
      certbot: 1
      update_policy:
      algorithm: "hmac-sha512"
      tsig_key: !vault |
         $ANSIBLE_VAULT;1.1;AES256
         37455546613735396364323363323464393731626466616262613033656264343765306238353934
         [...]
         38658654654542113212121124322324342223243032623430353765646639366536663566653836
         32643931393165643236

  test.local:
    - name: ddns_testlocal_certbot
      certbot: 1
      algorithm: "hmac-sha512"
      tsig_key: !vault |
         $ANSIBLE_VAULT;1.1;AES256
         37455546613735396364323363323464393731626466616262613033656264343765306238353934
         [...]
         38658654654542113212121124322324342223243032623430353765646639366536663566653836
         32643931393165643236
    - name: ddns_testlocal_edit
      algorithm: "hmac-sha512"
      tsig_key: !vault |
         $ANSIBLE_VAULT;1.1;AES256
         37455546613735396364323363323464393731626466616262613033656264343765306238353934
         [...]
         38658654654542113212121124322324342223243032623430353765646639366536663566653836
         32643931393165643236
```

* playbook

```yaml
- hosts: dns-server
  roles:
    - bind 
```

## Development
### Todo list

- [x] TSIG key management for DynDNS (eg: certbot RFC2136)
- [ ] Secure zones tranfers using TSIG 
- [ ] Master/Slave configuration
- [x] Clean symblinks creation for KSK/ZSK keys
- [ ] Remove `dnssec-keygen` calls and cron task
- [ ] Manage properly `journal out of sync with zone` errors


### Test with molecule and docker

* install [docker](https://docs.docker.com/engine/installation/)
* install `python3` and `python3-pip`
* install molecule and dependencies `pip3 install molecule 'molecule[docker]' docker ansible-lint testinfra yamllint`
* run `molecule test`

## License

```
Copyright (c) 2017 Adrien Waksberg

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
