# otobo-ideas: Just some notes OTOBO

OTOBO is an open source ticketing system derived from ((OTRS)) Community Edition.

Here be only some ramblings.

## Ideas

### Use Github Issues for bugtracking

Bugzilla is so 2000s. I talked to Stefan about that and he decided that Issues should be used. Thanks for the nice Brotzeit by the way! https://github.com/RotherOSS/doc-otobo-installation/issues/1 is the first real issue. https://github.com/RotherOSS/otobo/issues/1 had been inadvertently added.

### Tighten the requirements

* Define the supported Linux and FreeBSD distributions
* Find the minimal Perl version

The minimal version of Perl is now `5.24.0`. Say hi to the optional `postderef_qq` feature and the automatically activated `postderef` feature.

* Find the minimal MySQL version
* Find the minimal Postgres version
* Find the minimal Apache2 version
* Document the found versions
* Check the requirements

### cpanfile generation from otobo.CheckModules.pl

Added the option `--cpanfile`. All required modules are reported. The generated cpanfile is also tracked in git and available during installation. Features are supported.

### PSGI support

PSGI application in bin/psgi-bin/otobo.psgi using CGI::Emulate::PSGI.
rcp.pl as a direct PSGI application.

* Test debugging with `Plack::M:iddleware::CamelcadeDB`
* Test profiling with Devel::NYTProf5.30.2-buster

### Mojolicious

Add the feature mojo to the cpanfile and add a sample application to `bin/mojo-bin`.

### Docker

Based on the official Perl image 5.30.2-buster using Debian 10.
cpanm is already installed.
Separate containers for web, db, and daemon.

### Remove Support for Perl-Ex.

Removed.

### Other Ideas

* Perl::Critic
* Perl::Tidy
* add SQLite backend, might be useful for devel and testing
* add a Selection for MariaDB in installation process, otherwise same as MySQL
* use `XML::Compile::WSDL`for SOAP
* `Seraeal`insted of `Storable`

### Discarded Ideas

* Check the adaptions done in OFORK

Don't do this as the code for OFORK is note easily available.

## Development process

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

### Working with Git


#### Exclude dirs and files without .gitignore

* vim ~/.git/info/exclude 

#### Sync with upstream repository

See https://thoughtbot.com/blog/keeping-a-github-fork-updated

* remote add rotheross https://github.com/RotherOSS/otobo
* git fetch rotheross
* git checkout rel-10_0
* git rebase rotheross/rel-10_0
* git status
* git push

#### Create a branch locally and push it to Github

* git checkout -b issue-otobo9-query_cache_size
* git push --set-upstream origin issue-otobo9-query_cache_size

### Aliases

* alias otobo_start="sudo systemctl start apache2.service"
* alias otobo_status="sudo systemctl status apache2.service"
* alias otobo_restart="sudo systemctl restart apache2.service"
* alias otobo_stop="sudo systemctl stop apache2.service"
* alias otobo_mysql="mysql -u otobo otobo -p"

## SEE ALSO
 
 * http://otobo.de
 * [RotherOSS Github repositories](https://github.com/RotherOSS/otobo)
