# ansible_common
Ansible Common Roles

- [Common](#common)
- [Firewalld](#firewalld)
- [LDAP](#ldap)
- [phpLDAPAdmin](#ldapadmin)
- [PWM](#pwm) : self password management web app
- [PostgreSQL](#postgres) : Use CentOS Base yum version
- [Ruby](#ruby)
- [Tomcat](#tomcat)
- [Glassfish](#glassfish)
- [Maven](#maven)
- [http_proxy](#http_proxy)

# <a name="common">Common</a>

- Usage

  ```yaml
    roles:
      - ../ansible_common/common
  ```

- No Argument

- Abstract : Common settings for almost all servers.
  1. Enable epel
  1. yum update
  1. install basic commands. curl wget postfix etc.
  1. disable SELinux :-P
  1. open mdns (avahi) ports.

# <a name="firewalld">Firewalld</a>

- Usage

  ```yaml
    roles:
      - { role: ../ansible_common/firewalld, PORTS:["4848/tcp","8080/tcp"], SERVICES:["ldap"] }     
  ```

- Argument

| Argument     | Default value | Explanation |
|:-------------|:--------------|:------------|
|PORTS      |[] |these ports are opened for client computers|
|SERVICES   |[] |these services are opened for client computers|

- Abstract : make specified ports and services open

# <a name="ldap">LDAP</a>

- Usage

  ```yaml
    roles:
      - { role: ../ansible_common/ldap, ROOT_PWD: "9l!fe"}      
  ```

- Argument

| Argument     | Default value | Explanation |
|:-------------|:--------------|:------------|
|ROOT_PWD |secret ||
|DOMAIN   |example|DOMAIN must be the first dc of the SUFFIX.|
|SUFFIX   |dc=example,dc=com|base dn|
|ORG      |example co. ltd.|Organization name.|

- Abstract : Install OpenLDAP
  1. install openldap
  1. create cn=Manager,dc=example,dc=com / password=secret
  1. install schmeas
    - core : basic attribute. "cn", "ou", etc.
    - cosine : x500/COSINE tree figure data structure
    - inetorgperson : name, group, email, etc.
    - memberof overlay : auto resolve 'memberof attributes' of inetorgperson. For example :

      ```ldif
      # admin user
      dn: cn=ichiro,ou=People,dc=example,dc=com
      objectClass: inetOrgPerson
      cn: ichiro
      sn: suzuki
      userPassword: ichiro123

      # admin group
      dn: cn=admin,ou=Group,dc=example,dc=com
      objectClass: groupOfNames
      cn: admin
      member: cn=ichiro,ou=People,dc=example,dc=com
      ```

      ```shell
      $ ldapsearch -x -D "cn=Manager,dc=example,dc=com" -W -b "cn=ichiro,ou=People,dc=example,dc=com" memberof
      dn: cn=ichiro,ou=People,dc=example,dc=com
      memberOf: cn=admin,ou=Group,dc=example,dc=com
      ```
  1. create sample directory

    ```
      - dc=com
        - dc=example
          - Group (organizationalUnit)
            - admin (groupOfNames) ⇒ member = cn=ichiro,ou=People,dc=example,dc=com
          - People (organizationalUnit)
            - ichiro (inetOrgPerson) ⇒ userPassword=ichiro123
            - jiro (inetOrgPerson) ⇒ userPassword=jiro123
    ```
  1. set access authentication

| Attribute    | User        | Auth         |
|:-------------|:------------|:------------:|
| userPassword |cn=Manager,dc=example,dc=com|manage        |
|              |self         |write         |
|              |anonymous    |auth          |
|              |* (other)    |none          |
| * (other)    |cn=Manager,dc=example,dc=com|manage        |
|              |self         |write         |
|              |* (other)    |none          |

- Appendix. About typical attibutes of LDAP

| Attribute    | Note        |
|:-------------|:------------|
|dn            |a location of tree. Ex. cn=ichiro,ou=People,dc=example,dc=com|
|dc            |domain component. Ex. dc=example, dc=com|
|c             |country|
|o             |organization. Ex. a company|
|ou            |organization unit. Ex. a division in the company|
|cn            |lastname (family-name)|
|sn            |firstname (given-name)|
|uid           |user id|
|userPassword  |password. You shoud store hash value of password. Don't put plain text.|
|RDN           |primary key (attribute) during brother nodes|

- Appendix. How can I use groupOfUniqueNames as auth group objectClass instead of groupOfNames.
  - Settings for the memberof overlay is in cn=config. And default settings is following :
  ```
  olcMemberOfRefInt: TRUE
  olcMemberOfGroupOC: groupOfNames
  olcMemberOfMemberAD: member
  ```
  - So, to refer the uniqueMember attribute in the  groupOfUniqueNames objectClass when seeking inetOrgPerson, you must overwrite these config attribute.
  - more details, see [man page](http://www.openldap.org/software/man.cgi?query=slapo-memberof&apropos=0&sektion=0&manpath=OpenLDAP%202.4-Release&format=html)
  - But an ancient old wise man said you should not change the default setting in vain!

# <a name="ldapadmin">phpLDAPAdmin</a>

- Usage

  ```yaml
    roles:
      - ../ansible_common/ldapadmin
  ```

- No Argument

- Abstract
  1. install apache
  1. install phpLDAPAdmin

# <a name="pwm">PWM</a>

  - Usage

    ```yaml
      roles:
        - ../ansible_common/pwm
    ```

  - No Argument

  - Abstract
    1. install PWM to /opt/tomcat/webapps
    1. If there is apache, install mod_proxy_http settings.

# <a name="postgres">PostgreSQL</a>

  - Usage

    ```yaml
      roles:
        - { role: ../ansible_common/postgres, db_name: redmine, db_passwd: "{{ db_passwd_redmine }}" }
    ```

  - Argument

| Argument     | Default value | Explanation |
|:-------------|:--------------|:------------|
|db_name     |(none) |username for db|
|db_passwd   |(none) |password for db|

  - Abstract
    1. install postgresql (CentOS Base yum version, not latest)
    1. create user.
    1. create db that {{db_name}} user has full permission.

# <a name="ruby">Ruby</a>

- Usage

  ```yaml
    roles:
      - role: ../ansible_common/ruby
  ```

- Argument

| Argument     | Default value | Explanation |
|:-------------|:--------------|:------------|
|ruby_version  |ruby-2.3.1| source code is published in https://cache.ruby-lang.org/pub/ruby/2.3/{{ ruby_version }}.tar.gz|

- Abstract
  1. donwload ruby source
  1. build ruby
  1. install ruby

# <a name="tomcat">Tomcat</a>

- Usage

  ```yaml
    roles:
      - role: ../ansible_common/tomcat
  ```

- Argument

| Argument     | Default value | Explanation |
|:-------------|:--------------|:------------|
|JAVA_VERSION|1.8.0||
|TOMCAT_VERSION  |8.5.8| source code is published in http://ftp.tsukuba.wide.ad.jp/software/apache/tomcat/tomcat-8/v{{ TOMCAT_VERSION }}/bin/apache-tomcat-{{ TOMCAT_VERSION }}.tar.gz|

- Abstract
  1. install openjdk 1.8.0 and postgresql/mysql jdbc driver
  1. donwload tomcat source
  1. install tomcat
  1. make symlink of jdbc drivers to /opt/tomcat/lib/
  1. create systemd unit and enable it

# <a name="glassfish">Glassfish</a>

- Usage

  ```yaml
    roles:
      - { role: ../ansible_common/glassfish
          , db_name: gis
          , db_passwd: password
          , db: postgis
        }
  ```

- Argument

| Argument     | Default value | Explanation |
|:-------------|:--------------|:------------|
|JAVA_VERSION|1.8.0||
|GLASSFISH_VERSION  |4.1.2| source code is published in http://download.java.net/glassfish/{{ GLASSFISH_VERSION }}/release/glassfish-{{ GLASSFISH_VERSION }}.zip|
|ADMIN_PASSWORD|password||
|db_name||Database name (it is same as the database admin user)|
|db_password||password for {{db_name}}|
|db||Database type [postgis, postgresql]|

- Abstract
  1. install openjdk 1.8.0
  1. donwload glassfish source
  1. install glassfish
  1. create systemd unit and enable it
  1. create jms queue jms/myQueue
  1. create datasource jdbc/sample

- Glassfish 4.1.1 has bug at the management console in Nov-2016. see [JIRA GLASSFISH-21443](https://java.net/jira/browse/GLASSFISH-21443)
- JDBC Drivers would be installed.
- If you want to prepare the postgis jdbc driver for the glassfish, you must call the maven role before calling this role.
- The "jdbc/sample" refers the app-connection-pool. You can add another datasource for example "jdbc/activitiDS" that refers the app-connection-pool later. Or You can change the "jdbc/sample" to your favorite name.

# <a name="maven">Maven</a>

- Usage

  ```yaml
    roles:
      - role: ../ansible_common/maven
  ```

- Argument

| Argument     | Default value | Explanation |
|:-------------|:--------------|:------------|
|JAVA_VERSION |1.8.0||
|MAVEN_VERSION|3.3.9||

- Abstract
  1. install openjdk 1.8.0
  1. install maven 3.3.9 (yum version is too old, install from apache site)
  1. create .m2/settings.xml if proxy.yml is defined.

- if proxy settings is needed, write like follows :

```yaml
---
http_proxy_host: proxy.foo.com
http_proxy_port: 3128
http_proxy_username: john@foo.com
http_proxy_password: password

http_proxy: http://{{ http_proxy_username | urlencode() }}:{{ http_proxy_password }}@{{ http_proxy_host }}:{{ http_proxy_port }}

proxy_env :
  no_proxy: 127.0.0.1,localhost
  http_proxy: "{{ http_proxy | default(None) }}"
  https_proxy: "{{ http_proxy | default(None) }}"
```
# <a name="http_proxy">http_proxy (reverse proxy settings for apache)</a>

- Usage

  ```yaml
    roles:
      - { role: ../ansible_common/http_proxy, REQUEST:"/pwm/", DEST:"http://localhost:8080/pwm/"}
  ```

- Argument

| Argument     | Default value | Explanation |
|:-------------|:--------------|:------------|
|REQUEST |(none)|If user requests url which has {{ REQUEST }}, http_proxy will farm out the request to {{ DEST }}|
|DEST    |(none)||
