# Just some notes about [OTOBO](https://otobo.de/en/community/)

OTOBO is an open source ticketing system that was derived from ((OTRS)) Community Edition.

Here are some ramblings.

## Ideas

### Open Ideas

Some ideas I'm tinkering with.

* Database
  * add SQLite backend, might be useful for devel and testing
  * add H2 backend, interesting because of the compatability modes
  * use DBD::MariaDB instead of DBD::mysql
* Implementation
  * use `XML::Compile::WSDL`for SOAP
  * use `Sereal`insted of `Storable`
  * use `Email::Sender`
  * use `Log::Log4perl`
  * use `Capture::Tiny`
* Testing
  * currently about 35 Selenium tests are still failing
  * maybe use Plack::Test
* Continous Integration
  * pretty much still TODO
* Translation
  * use N-tier translations, e.g. Portuguese falling back to Brazilian Portugues falling back to English
  * A Louisiana Crèole translation
* Web related
  * Mojolicious or Raisin, or .., for the Generic Interface
  * Websockets
  * Check whether OpenGraph is supported
  * facicon 
  * Eliminate Apache altogether
* Security
  * Find a maintained source of CPAN security advisories, like https://github.com/vti/cpan-security-advisory  

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

## PSGI

PSGI is now officially supported. But of course things can always be improved.

### PSGI support improvements

* Make better use of middlewares

* Saner support for webservices like SOAP, REST 

* Plack::Test

* Streaming, chat server

### PSGI stumbling blocks

- `%ENV`no longer has the accustomed data. The Request-Object in `Kernel::System::Web::Request` must be used for that.
- Encoding issues might be lurking. But the gut feeling is that things are more simple with PSGI.
- Care must be taken that HTTP headers are not set in the content. `Kernel::System::Web::Response` must be used.
- There are dependencies on CPAN modules.
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

## Refreshing modules

One feature of OTOBO is that in a persistent Perl environment modules on disk might change and therefore must be reloaded.
The use cases are:

- An OTOBO package overwrites an existing module

- An OTOBO package creates a new version of a module in the _Custom_ folder

- A config cache was updated in _Kernel/Config/Files
  - ZZZAAuto.pm
  - ZZZACL.pm
  - ZZZProcessManagement.xml
  - possibly others

### Current implementation for using OTOBO 10.0.9 under Apache

It is required that mod_perl is activated and that Apache2::Reload is installed. The Apache config has these settings:

    # Reload Perl modules when changed on disk
    PerlModule Apache2::Reload
    PerlInitHandler Apache2::Reload
    
This means that for every request the modification time all modules in `%INC` are checked. Modified modules are reloaded.

`Kernel::System::ObjectManager::ObjectsDiscard` has the option `ForcePackageReload` which removes the module from `%INC`. However it looks like this option is almost never used.

`Kernel::System::MigrateFromOTRS::Base::RebuildConfig()` and `Kernel::System::Daemon::DaemonModules::SchedulerTaskWorker::Run` remove _ZZZAAuto.pm_ and _ZZZAuto.pm_ from `%INC` in order to force are reload of the config cache.

`Kernel::System::SysConfig::Base::OTOBOCommunity::UserConfigurationDeploySync` also deletes from `%INC`. Not sure whether this is still relevant.

`Kernel::System::ZnunyHelper::RebuildConfig` deletes from `%INC`.



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
