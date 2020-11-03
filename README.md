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

```
alias otobo_apache_start="sudo systemctl start apache2.service"
alias otobo_apache_status="sudo systemctl status apache2.service"
alias otobo_apache_restart="sudo systemctl restart apache2.service"
alias otobo_apache_stop="sudo systemctl stop apache2.service"
alias otobo_mysql="mysql -u otobo otobo -p"
alias otobo_docker_login_web="docker exec -it otobo_web_1 bash"
alias otobo_docker_login_db="docker exec -it otobo_db_1 bash"
alias otobo_docker_login_daemon="docker exec -it otobo_daemon_1 bash"
alias otobo_docker_login_redis="docker exec -it otobo_redis_1 sh"
alias otobo_docker_login_nginx="docker exec -it otobo_nginx_1 bash"
alias otobo_docker_login_elastic="docker exec -it otobo_elastic_1 bash"
alias otobo_docker_update_local-rel-10_0="docker run -it --rm --volume otobo_opt_otobo:/opt/otobo otobo:local-rel-10_0 update"
alias otobo_docker_update_local-rel-10_1="docker run -it --rm --volume otobo_opt_otobo:/opt/otobo otobo:local-rel-10_1 update"
alias otobo_docker_update_10_0="docker run -it --rm --volume otobo_opt_otobo:/opt/otobo rotheross/otobo:devel update"
alias otobo_docker_update_10_1="docker run -it --rm --volume otobo_opt_otobo:/opt/otobo rotheross/otobo:devel-10_1 update"
alias otobo_docker_quick_setup="docker exec -t otobo_web_1 bash -c \"date ; hostname ; rm -f Kernel/Config/Files/ZZZAAuto.pm ; bin/docker/quick_setup.pl --db-password otobo_root\""
alias otobo_docker_test_suite="docker stop otobo_daemon_1 ; docker exec -t otobo_web_1 bash -c \" date ; hostname ; bin/docker/run_test_suite.sh\" ; docker start otobo_daemon_1"
alias otobo_docker_test_progress="docker exec -t otobo_web_1 bash -c \" date ; ls -l prove_*.out ; wc -l prove_*.out ; grep '^not ok ' prove_*.out | grep -v -c '# TODO'\""
alias otobo_docker_backup="docker run -it --rm --volume otobo_opt_otobo:/opt/otobo --volume otobo_backup:/otobo_backup --network otobo_default otobo:local scripts/backup.pl -d /otobo_backup"
alias otobo_docker_restore="docker run -it --rm --volume otobo_opt_otobo:/opt/otobo --volume otobo_backup:/otobo_backup --network otobo_default otobo:local scripts/restore.pl -d /otobo_backup"
alias otobo_prove="prove -I . -I Kernel/cpan-lib -I Custom --verbose -r"
alias otobo_perl="perl -I . -I Kernel/cpan-lib -I Custom"
```

## SEE ALSO
 
 * http://otobo.org
 * [RotherOSS Github repositories](https://github.com/RotherOSS/otobo)
