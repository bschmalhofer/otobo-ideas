# Just some notes about [OTOBO](https://otobo.de/en/community/)

OTOBO is an open source ticketing system that was derived from ((OTRS)) Community Edition.

Here are some ramblings.

## Ideas

### Ideas for OTOBO 11.0

* use `Capture::Tiny` https://github.com/RotherOSS/otobo/discussions/1042
* use `Feature::Compat::Try`https://github.com/RotherOSS/otobo/issues/1695
* use `Cpanel::JSON::XS`. https://github.com/RotherOSS/otobo/issues/399
* Using Regexp::Common and Regexp::Grammar
* use XML::LibXSLT
* DBD::MariaDB instead of DBD::MySQL https://github.com/RotherOSS/otobo/issues/1860
* DB: last_insert_id(), fetch-Loop
* Some kind of OTOBO Middleware: https://github.com/RotherOSS/otobo/issues/1617
* require Perl 5.26 released in 2017
    * https://metacpan.org/release/XSAWYERX/perl-5.26.0/view/pod/perldelta.pod
    * indented heredocs: nice feature IMHO
    * lexical subroutines: Should they be used?
    * Somewhat unreleated: What about restricted hashes?
* Support for `sd_notify()` in the OTOBO Daemon
* document deprecations and support policy: https://github.com/RotherOSS/otobo/issues/1204
* Allow bypass of ObjectManager, e.g. Kernel::System::DateTime, https://github.com/RotherOSS/otobo/discussions/1112
* Streaming, https://github.com/RotherOSS/otobo/discussions/1348
* Cool URIs. https://github.com/RotherOSS/otobo/discussions/1599, https://github.com/RotherOSS/otobo/issues/116, https://github.com/RotherOSS/otobo/issues/1590
* Remove example config for FastCGI: https://github.com/RotherOSS/otobo/issues/132
* Simplify SELECTS. https://github.com/RotherOSS/otobo/issues/1916
* less magic in Kernel::System::Main::Dump(), https://github.com/RotherOSS/otobo/issues/694

### Open Ideas

Some ideas I'm tinkering with.

* Database
  * add SQLite backend, might be useful for devel and testing
  * add H2 backend, interesting because of the compatability modes
  * use DBD::MariaDB instead of DBD::mysql
* Implementation
  * use `XML::Compile::WSDL11`for SOAP
  * use `Sereal`instead of `Storable`
  * use `Mail::Message and Mail::Transport`
  * use `Log::Log4perl`
  * use `CHI`
  * Enhance import statements: https://metacpan.org/pod/perlimports
* Testing
  * maybe use Plack::Test
  * check NYTProf again
  * Devel::Cover
* Translation
  * use N-tier translations, e.g. Portuguese falling back to Brazilian Portugues falling back to English
  * A Louisiana Crèole translation
* API related
  * specification as an OpenAPI document
  * generate stubs, maybe for Raisin
  * implement the backend  
* Web related
  * Websockets
  * Check whether OpenGraph is supported
  * Eliminate Apache altogether
  * document how run OTOBO with plackup, without Docker
  * Switch from REST::Client to Mojo::UserAgent in the generic interface
  * Support for status 103 though the code is still experimental: https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/103, https://metacpan.org/pod/Gazelle#psgix.informational
* Devops
  * provide unit files for systemd 
* Security
  * Update CPAN::Audit
* File system
  * When articles are stored in the file system the a directory tree with a lot of files is possible. This is a pain for backup. How about transfering old files to squashfs or such?
* Events
  * Check whether the AsynchronousExecutor is good enough 
  * Alternatively switch to a real event queue like ActiveMQ or Minion  

### Implemented, or Discarded, Ideas

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

* The minimal version of Perl is now `5.24.0`. 

  Say hi to the optional `postderef_qq` feature and the automatically activated `postderef` feature.

* Minimal Apache Version defined as 2.4.

  Hopefully gone altogether soon.

* Support for Perl-Ex is removed

* Define, document and check the minimal supported  Linux and FreeBSD distributions

* Define, document and check the minimal supported  MySQL version
  
* Define, document and check the minimal supported  PostgreSQL version
 
* Perl::Critic, part of the CodePolicy
 
* Perl::Tidy, part of the CodePolicy

* Test scripts can use `Test2::V0` and `Kernel::System::UnitTest` side by side

* favicon not needed as shortcut icon is set in HTMLHead.tt

* Changed files in _Kernel/Config/Files_ are reloaded in `Kernel::Config:Defaults::new()`

* In OTOBO 10.1 HTTP/2 is supported by the Nginx TLS terminating proxy

## PSGI

PSGI is now officially supported. But of course things can always be improved.

### PSGI support improvements

* Make better use of middlewares

* Saner support for webservices like SOAP, REST 

* Plack::Test

* Streaming, chat server

### PSGI stumbling blocks

Things that must be considered when porting OTOBO core or OTOBO modules to 10.1.x.

- `%ENV`no longer has the accustomed data. The Request-Object in `Kernel::System::Web::Request` must be used for that.
- Encoding issues might be lurking. But the gut feeling is that things are more simple with PSGI.
- Care must be taken that HTTP headers are not set in the content. `Kernel::System::Web::Response` must be used.
- There are dependencies on CPAN modules.
- Redirects must be done by calling K::O::H::Layout::Redirect()
- Fatal errors muse be thrown by calling K::O::H::Layout::FatalError()
- Printing to STDOUT is no longer a thing.

### What about CGI scripts ?

The scripts in _bin/cgi-bin_ are now wrapping otobo.psgi.

### What about Mojolicious ?

Mojolicious itself is not based on PSGI. However there are adaptors that make a Mojo app behave like a PSGI app.
Before migrating to Mojolicious there should be an analysis which benefits of Mojo warrant a migration.
The PSGI interface is more simple and consistent than CGI. Therefore a migration to Mojo is more simple from a PSGI app than from a CGI app.I

It would be interesting to see how OTRS uses Mojolicious.

### What about Kelp ?

Kelp is a thin wrapper around Plack. This makes it IMHO an interesting option for OTOBO.

### Current status

Deep support for PSGI in the 10.1 branch

## Helper, Tips and Tricks

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

### Git prompt

```
if [ "$color_prompt" = yes ]; then
    #PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
    PS1='${debian_chroot:+($debian_chroot)}bes:\[\033[01;34m\]\w\[\033[00m\] \[\033[01;32m\]$(__git_ps1 "(%s)")\[\033[00m\]\$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
fi
```

### Shell helpers

* https://github.com/rupa/z

### Automatic Editing

```
find scripts/test/ -name '*.t' | xargs -L 1 perl -0777 -pi -e 's/\n\n\n\$Self->DoneTesting\(\);/\n\n\$Self->DoneTesting();/'
```

## SEE ALSO
 
 * http://otobo.org
 * [RotherOSS Github repositories](https://github.com/RotherOSS/otobo)
 * https://github.com/RotherOSS/otobo/issues
 * https://metacpan.org/pod/distribution/PSGI/PSGI/FAQ.pod
