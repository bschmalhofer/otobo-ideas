# otobo-ideas: Just some notes OTOBO

OTOBO is an open source ticketing system which was derived from ((OTRS)) Community Edition.

Here be only some ramblings.

## Ideas

### Open Ideas

Some ideas I'm tinkering with.

* add SQLite backend, might be useful for devel and testing
* use DBD::MariaDB instead of DBD::mysql
* use `XML::Compile::WSDL`for SOAP
* use `Sereal`insted of `Storable`
* use `Email::Sender`
* use `Log::Log4perl`
* use N-tier translations, e.g. Portuguese falling back to Brazilian Portugues falling back to English
* A Louisiana Crèole translation
* Eliminate Apache

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

A sample had been done in /otobo/dbviewer. Removed in OTOBO 10.0.9.

* Docker

Done, see https://hub.docker.com/repository/docker/rotheross/otobo.

* The minimal version of Perl is now `5.24.0`. Say hi to the optional `postderef_qq` feature and the automatically activated `postderef` feature.

* Minimal Apache Version defined as 2.4.

Hopefully gone altogether soon.

* Support for Perl-Ex is removed

* Define, document and check the minimal supported  Linux and FreeBSD distributions

* Define, document and check the minimal supported  MySQL version
  
* Define, document and check the minimal supported  PostgreSQL version
 
* Perl::Critic, part of the CodePolicy
 
* Perl::Tidy, part of the CodePolicy

## Ramblings on PSGI

An interesting read is https://metacpan.org/pod/distribution/PSGI/PSGI/FAQ.pod

### Why PSGI ?

Reasons why I thing that PSGI is the way to go.

* It's more modern.

* Better abstraction. No need to get into the details of Apache+mod_perl.

* Middlewares can be used to reduce the complexity.

* Configuration is more simple. _apache2-perl-startup.pl_ is no longer needed.

* Current development on the Perl community is often based on PSGI. See the first bullet point.

* Convenience. It's convienient the have sane interfaces, sane request and response objects.

* CGI.pm is a beast. Plack::Request would most likely be a better choice.

* Easy webserver integration: Perl webservers, Apache+mod_perl, FastCGI

* Easily pluggable into existing infrastructure

* Easy plugging of other applications.

* In future: more simple support for webservices, SOAP, REST 

* Plack::Test is nice too

* Finally there is streaming

### How deep should PSGI support be ?

That's a real question. With CGI::Emulate::PSGI, one can take the old application, wrap it and conform to the
PSGI interface. I rather go for a real migration, where the intended interfaces are used. The `%Env`has for the input, and
an array ref as the response.

### What are the stumbling block?

These points are specific for a deep migration.

- The `%ENV`no longer has the accostumed data. The CGI-Object in `Kernel::System::Web::Request` must be used for that.
- Encoding issues might be lurking. But the gut feeling is that things are more simple with PSGI.
- Care must be taken that HTTP headers are not set in the content. `Kernel::System::Web::Response` must be used.
- More dependencies on CPAN modules.

### Why no traditional CGI scripts ?

The scripts in _bin/cgi-bin_ are no longer needed because `PerlResponseHandler Plack::Handler::Apache2` can be used for making _otobo.psgi_ available in Apache.
There might be a reason to still provide the scripts. In this case `Plack::Handler::CGI` can be used. 

The real reason for getting rid of the scripts is to reduce the maintainance effort. Having two ways of generating content increases the burden on testing. The code can be simplified a little bit when only PSGI must be converted.

### What about Mojolicious ?

Mojolicious itself is not based on PSGI. However there are adaptors that make a Mojo app behave like a PSGI app.
Before migrating to Mojolicious there should be an analysis which benefits of Mojo warrant a migration.
The PSGI interface is more simple and consistent than CGI. Therefore a migration to Mojo is more simple from a PSGI app than from a CGI app.I

It would be interesting to see how OTRS uses Mojolicious.

### What about Kelp ?

Kelp is a thin wrapper around Plack. This makes it IMHO an interesting option for OTOBO.

### Solution

Deep support for PSGI in the 10.1 branch

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

### Working with IntelliJ IDEA

TODO

### Aliases

These are the bash aliases I use, or at least have set up:
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
alias otobo_docker_login_selenium="docker exec -it otobo_selenium-chrome_1 bash"
alias otobo_docker_login_elastic="docker exec -it otobo_elastic_1 bash"
alias otobo_docker_update_local-rel-10_0="docker run -it --rm --volume otobo_opt_otobo:/opt/otobo otobo:local-rel-10_0 update"
alias otobo_docker_update_local-rel-10_1="docker run -it --rm --volume otobo_opt_otobo:/opt/otobo otobo:local-rel-10_1 update"
alias otobo_docker_update_10_0="docker run -it --rm --volume otobo_opt_otobo:/opt/otobo rotheross/otobo:devel update"
alias otobo_docker_update_10_1="docker run -it --rm --volume otobo_opt_otobo:/opt/otobo rotheross/otobo:devel-10_1 update"
alias otobo_docker_quick_setup="docker exec -t otobo_web_1 bash -c \"date ; hostname ; rm -f Kernel/Config/Files/ZZZAAuto.pm ; bin/docker/quick_setup.pl --db-password otobo_root --http-port 81\""
alias otobo_docker_test_suite="docker stop otobo_daemon_1 ; docker exec -t otobo_web_1 bash -c \" date ; hostname ; bin/docker/run_test_suite.sh\" ; date ; docker start otobo_daemon_1"
alias otobo_docker_test_progress="docker exec -t otobo_web_1 bash -c \" date ; ls -l prove_*.out ; wc -l prove_*.out ; grep '^not ok ' prove_*.out | grep -v -c '# TODO'\""
alias otobo_docker_backup="docker run -it --rm --volume otobo_opt_otobo:/opt/otobo --volume otobo_backup:/otobo_backup --network otobo_default otobo:local scripts/backup.pl -d /otobo_backup"
alias otobo_docker_restore="docker run -it --rm --volume otobo_opt_otobo:/opt/otobo --volume otobo_backup:/otobo_backup --network otobo_default otobo:local scripts/restore.pl -d /otobo_backup"
alias otobo_prove="prove -I . -I Kernel/cpan-lib -I Custom --verbose -r"
alias otobo_perl="perl -I . -I Kernel/cpan-lib -I Custom"
```

### Shell helpers

* https://github.com/rupa/z


## SEE ALSO
 
 * http://otobo.org
 * [RotherOSS Github repositories](https://github.com/RotherOSS/otobo)
 * https://github.com/RotherOSS/otobo/issues
