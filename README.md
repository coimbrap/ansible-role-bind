# Ansible role: Bind

Install and configure bind with dnssec

- Forked by Mablr for Elukerio (work on DNSSEC & TSIG)
- Forked by Pierre C to work on views and master/slave

## Requirements

* Ansible >= 2.7
* bind version >= 9.8
* Debian Stretch and Buster

## Role variables

* `bind_options` - hash general bind options
* `bind_views` - the dns views (contain zones)
* `bind_acl` - the dns acl
* `bind_dnssec` - hash with dnssec configuration
* `bind_tsig` - the tsig keys
* `bind_listen_ipv4` - enable or disable ip v4 support (default: true)
* `bind_listen_ipv6` - enable or disable ip v6 support (default: true)
## How to use

* `group_vars/dns-server/bind`

```yaml
bind_options:
  server-id: '"1"'
  bind_role: master
  bind_views:
    ext:
      recursion: none
      acl: any
      zones:
        - devo.re:
            reverse: false
            ns_primary: ns-a.devo.re
            mail: root@ns-a.devo.re
            options:
              auto-dnssec: maintain
              inline-signing: yes
            records:
              - { name: '@', type: ns, value: ns-a.devo.re. }
              - { name: '@', type: a, ttl: 5m, value: 195.154.163.18 }
              - { name: '*', type: a, ttl: 5m, value: 195.154.163.18 }
    int:
      recursion: any
      acl: int
      zones:
        - devo.re:
            reverse: true
            ipv4: 10.0.20.1/24
            ns_primary: ns-a.devo.re
            mail: root@ns-a.devo.re
            options:
              auto-dnssec: maintain
              inline-signing: yes
            records:
              - { name: '@', type: ns, value: ns-a.devo.re. }
              - { name: 'ns-a', type: a, ttl: 5m, value: 10.0.20.105 }
              - { name: 'pve', type: a, ttl: 5m, value: 10.0.20.1 }
              - { name: 'opn', type: a, ttl: 5m, value: 10.0.20.254 }
              - { name: 'ldap', type: a, ttl: 5m, value: 10.0.20.106 }
```

* `group_vars/dns-server/dnssec.yml`

```yaml
bind_dnssec:
  test.local:
    ksk:
      algorithm: 13
      digest: 3
      tag: 27950
      public_key: AMLbvPb3oQAW4CRXupC5nlpR+0IG/9rOpQ0yNMkYNn8FzsAlxAFKkTURzDBYI7ZEKwQbWXfzGt43N9lh7yB57A==
      private_key: !vault |
        $ANSIBLE_VAULT;1.1;AES256
        33373964393565343638363964366133663235653931386664343435326362333031323130363362
        [...]
        65616337363634636365386166643133373331336333376430353663303563346236316532336532
        62376530646231346237
      zsk:
        algorithm: 13
        digest: 3
        tag: 9897
        public_key: Nmtx436O2y8qRhzTgzC7INhxoMsy4z7QzJWqYnYAqd/6fJgsxMGRNwDIHs+xuSi1Zav0liOytIJzvC+VXJcEUg==
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
  transfer:
    - name: tsigkey_global
      zone: wild
      algorithm: "hmac-sha512"
      secret:  !vault |
         $ANSIBLE_VAULT;1.1;AES256
         37455546613735396364323363323464393731626466616262613033656264343765306238353934
         [...]
         38658654654542113212121124322324342223243032623430353765646639366536663566653836
         32643931393165643236

  edition:
    - name: ddns_devore_adm
      zone: devo.re
      policy: custom
      policy_custom: zonesub any
      algorithm: "hmac-sha512"
      secret:  !vault |
         $ANSIBLE_VAULT;1.1;AES256
         37455546613735396364323363323464393731626466616262613033656264343765306238353934
         [...]
         38658654654542113212121124322324342223243032623430353765646639366536663566653836
         32643931393165643236
    - name: ddns_certbot_global
      policy: certbot
      zone: wild
      algorithm: "hmac-sha512"
      secret: !vault |
         $ANSIBLE_VAULT;1.1;AES256
         37455546613735396364323363323464393731626466616262613033656264343765306238353934
         [...]
         38658654654542113212121124322324342223243032623430353765646639366536663566653836
         32643931393165643236
```

* `group_vars/dns-server/acl`

```yaml
bind_acl:
  int:
    - "10.0.20.0/24"
    - "localhost"
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
- [x] Secure zones tranfers using TSIG
- [x] Clean symblinks creation for KSK/ZSK keys
- [x] Remove `dnssec-keygen` calls and cron task
- [x] Manage properly `journal out of sync with zone` errors
- [ ] Master/Slave configuration
- [x] Multiple views management
- [ ] Clean dns reverse


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
