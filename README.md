# Role netbox

Role nasadí aplikaci NetBox a její závislosti (webserver, postgresql a redis).

Konfiguraci redis předpokládáme takovou (viz role redis), že server poslouchá
na `localhost:6379`.

NetBox momentálně **není** připraven na běh v HA režimu.

Konfigurace je silně inspirována [dokumentací](https://netbox.readthedocs.io/en/stable/installation/).
Jediné, co je třeba udělat ručně, je vytvoření super uživatele:

```bash
/srv/www/netbox/venv/bin/python3 /srv/www/netbox/netbox/manage.py createsuperuser --noinput --username root --email superadmin@logicworks.cz
```

## Aktualizace

Ansible role umožňuje aktualizace sazené NetBox aplikace nastavení proměnné `update`
na hodnotu `true`. Na příkazové řádce: `-e update=true`.


## Proměnné

Konfigurace `netbox` se děje pomocí slovníku `netbox`.

| Proměnná            | Povinná | Výchozí            | Popis |
| ------------------- | ------- | ------------------ | ----- |
| user                | ne      | netbox             | Systémový uživatel, pod kterým poběží systemd služba |
| group               | ne      | netbox             | Skupina (uživatele), pod kterou poběží systemd služba |
| groups              | ne      |                    | Další skupiny, kterých bude `user` členem |
| repo                | ne      | `https://github.com/netbox-community/netbox.git` | Git repozitář projektu NetBox |
| version             | ne      | master             | Verze NetBox, kterou chceme nasadit (git tag nebo větev) |
| install_dir         | ne      | /srv/www/netbox    | Adresář, kam se aplikace nainstaluje |
| changelog_retention | ne      | 365                | Doba (dny), po kterou NetBox drží v databázi všechny změny v aplikaci |
| secret_key          | ano     |                    | Tajný klíč pro šifrování secrets. Aspoň 50 znaků |
| plugins             | ne      |                    | Slovník pro konfiguraci pluginů |
| name                | ano     |                    | Název pluginu, jak jem používá NetBox |
| package_name        | ano     |                    | Název pluginu, jak jej používá PIP |
| config              | ne      |                    | RAW Python konfiguraci pluginu |
|                     |         |                    |      |
| redis_password      | ano     |                    | Heslo do lokální instance redis serveru |
| redis_enable_tcp    | ano     | false              | Nutné zapnout |
|                     |         |                    |      |
| database            | ano     |                    | Slovník pro nastavení přístupu do PostgreSQL databáze |
| .name               | ne      | netbox             | Jméno databáze |
| .user               | ne      | netbox             | Jméno databázového uživatele pro přístup k databázi |
| .password           | ano     |                    | Heslo databázového uživatele |
|                     |         |                    |      |
| ldap                | ne      |                    | Slovník pro nastavení integrace s LDAP serverem |
| .enabled            | ne      | false              | Zapnutí LDAP integrace |
| .starttls           | ne      | true               | Zapnutí režimu připojení `STARTTLS` na portu 389 |
| .base_dn_users      | ano     |                    | DN pod kterým se vyhledávají uživatelé |
| .user_dn_template   | ano     |                    | DN template pro přímý přístup k objektu uživatele, aby nemusel být vyhledán |
| .base_dn_groups     | ano     |                    | DN pod kterým se vyhledávají skupiny |
| .group_require      | ne      |                    | LDAP skupina, jejíž musí být uživatel členem, aby se mohl přihlásit |
| .group_mapping      | ne      |                    | Slovník pro mapování LDAP skupiny na netbox role |
| ..active            | ne      |                    | Uživatelé jsou aktivní a mohou se přihlásit |
| ..staff             | ne      |                    | Uživatelé mohou provádět změny |
| ..superuser         | ne      |                    | Uživatelé mají přístup do admin rozhraní |
| .user_attr_mapping  | ne      |                    | Slovník pro mapování atributů uživatele |
| ..first_name        | ne      | givenName          | LDAP atribut pro křestní jméno |
| ..last_name         | ne      | givenName          | LDAP atribut pro příjmení |
| ..email             | ne      | mail               | LDAP atribut pro e-mail |
|                     |         |                    |      |
| mail                | ne      |                    | Slovník pro nastavení integrace s mail serverem |
| .server             | ne      | localhost          | Adresa mail serveru |
| .port               | ne      | 25                 | SMTP port mail serveru |
| .tls                | ne      | none               | Zapne TLS Šifrování. Může být nastaveno v režimu 'tls' nebo 'starttls' |
| .username           | ne      |                    | Uživatel pro SMTP autentizaci |
| .password           | ne      |                    | Heslo pro SMTP autentizaci |
|                     |         |                    |      |
| vhost               | ne      |                    | Slovník s parametry pro apache virtual host |
| .name               | ne      | netbox             | Identifikátor vhosta (použitý pro jména souborů) |
| .site_name          | ne      | netbox.example.com | ServerName virtual hosta |
| .aliases            | ne      |                    | Pole aliasů virtual hosta |
| .admin_mail         | ne      | {{ admin_mail }}   | Email na admina webu |
| .https_enabled      | ne      | false              | Zapne HTTPS |
| .https_letsencrypt  | ne      | false              | Zapne získávání LE certifikátů |
| .https_redirect     | ne      | false              | Zapne Redirect HTTP -> HTTPS |
| .cert               | ne      | snakeoil.pem       | Cesta k certifikátu virtual hosta |
| .key                | ne      | snakeoil.key       | Cesta ke klíči virtual hosta |
| .chain              | ne      |                    | Cesta k souboru s certifikáty mezilehlých autorit |

## Příklady použití

```yaml
netbox:
  secret_key: 'drakdrakdrak...'
  database:
    password: 'drakdrak'
  ldap:
    enabled: true
    url: 'ldap://ldap.logicworks.cz'
    username: 'cn=user,ou=users,dc=logicworks,dc=cz'
    password: 'drakrdak'
    base_dn_users: 'ou=people,o=logicworks,dc=logicworks,dc=cz'
    user_dn_template: 'uid=%(user)s,ou=people,o=logicworks,dc=logicworks,dc=cz'
    base_dn_groups: 'ou=groups,o=logicworks,dc=logicworks,dc=cz'
    group_mapping:
      active: 'cn=active,ou=groups,o=logicworks,dc=logicworks,dc=cz'
      staff: 'cn=staff,ou=groups,o=logicworks,dc=logicworks,dc=cz'
      superuser: 'cn=superuser,ou=groups,o=logicworks,dc=logicworks,dc=cz'
  vhost:
    site_name: 'netbox.logicworks.cz'
    https_enabled: true
    https_letsencrypt: true
    https_redirect: true
```
