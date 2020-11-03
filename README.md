# otobo-ideas: Just some notes OTOBO

OTOBO is an open source ticketing system derived from ((OTRS)) Community Edition.

Here be only some ramblings.

## Ideas

### Open Ideas

* Define, document and check the minimal supported  Linux and FreeBSD distributions
* Define, document and check the minimal supported  MySQL version
* Define, document and check the minimal supported  PostgreSQL version
* Perl::Critic
* Perl::Tidy
* add SQLite backend, might be useful for devel and testing
* use DBD::MariaDB instead of DBD::mysql
* use `XML::Compile::WSDL`for SOAP
* use `Sereal`insted of `Storable`

### Discarded Ideas

* Check the adaptions done in OFORK

Don't do this as the code for OFORK is note easily available.

### Implemented Ideas

* Use Github Issues for bugtracking

Done, see https://github.com/RotherOSS/otobo/issues/.

* cpanfile generation from otobo.CheckModules.pl

Done. Even use Carton to some extent.

* PSGI support

Done, see otobo.psgi.

* Mojolicious

A sample is done, see /otobo/dbviewer. (Currently broken).

* Docker

Done, see https://hub.docker.com/repository/docker/rotheross/otobo.

* The minimal version of Perl is now `5.24.0`. Say hi to the optional `postderef_qq` feature and the automatically activated `postderef` feature.

* Minimal Apache Version defined as 2.4.

* Support for Perl-Ex is removed

## Ramblings on PSGI

### Why PSGI ?

### How deep should PSGI support be ?

### Why no tradiotional CGI scripts ?

### What about Mojolicious ?

### What about Kelp ?

## Notes regarding the development process

### Working with a git checkout on the local machine

* https://github.com/bschmalhofer/otobo is a fork of https://github.com/RotherOSS/otobo
* in a workspace as otobo: git clone https://github.com/bschmalhofer/otobo.git
* Link /opt/otobo to the checked out repos
* sudo /opt/otobo/bin/otobo.SetPermissions.pl
* Add local user to the group www-data, this should give you write privs
* Edit the code e.g. with IntelliJ IDEA
* check in the changed files as otobo
* push as otobo to Github
* Create pull request in Github

### Working with a git checkout in a VM

TODO

### Working with IntelliJ IDEA

TODO

### Aliases

* alias otobo_start="sudo systemctl start apache2.service"
* alias otobo_status="sudo systemctl status apache2.service"
* alias otobo_restart="sudo systemctl restart apache2.service"
* alias otobo_stop="sudo systemctl stop apache2.service"
* alias otobo_mysql="mysql -u otobo otobo -p"
* alias otobo_docker_login_web="docker exec -it otobo_web_1 bash"
* alias otobo_docker_login_db="docker exec -it otobo_db_1 bash"
* alias otobo_docker_login_cron="docker exec -it otobo_cron_1 bash"
* alias otobo_ssh_vo85="ssh vo85@vo85.vo.otrs.ch -p 22222"
* alias otobo_sshfs_vo85="sshfs -p 22222 vo85@vo85.vo.otrs.ch:/home/VirtualOTRS/vo85 fuse/"
* alias otobo_cpanm="cpanm --with-feature=mysql --with-feature plack --with-feature=mojo --installdeps ."


## SEE ALSO
 
 * http://otobo.de
 * [RotherOSS Github repositories](https://github.com/RotherOSS/otobo)
