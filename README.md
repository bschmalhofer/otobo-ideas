# otobo-ideas: Just some notes OTOBO

OTOBO is an open source ticketing system derived from ((OTRS)) Community Edition.

Here be only some ramblings.

## Ideas

### Use Github Issues for bugtracking

Bugzilla is so 2000s. I talked to Stefan about that and he decided that Issues should be used. Thanks for the nice Brotzeit by the way! https://github.com/RotherOSS/doc-otobo-installation/issues/1 is the first real issue. https://github.com/RotherOSS/otobo/issues/1 had been inadvertently added.

### Tighten the requirements

* Define the supported Linux and FreeBSD distributions
* Find the minimal Perl version
* Find the minimal MySQL version
* Find the minimal Postgres version
* Find the minimal Apache2 version
* Document the found versions
* Check the requirements

### cpanfile generation from otobo.CheckModules.pl

Added the option `--cpanfile`. All required modules are reported. 

TODO: Support for feature sets.

### PSGI support

bin/cgi-bin/app.psgi already exists and looks sensible.

* Check usage of SCRIPT_NAME in OTOBO
' Test debugging with `Plack::M:iddleware::CamelcadeDB`
* Convert some of the wrapped scripts to a real PSGI app
* Check the Apache-conf whether relevant features are missing.
* Check support for Devel::NYTProf

### Other ideas

* Docker image, see https://github.com/complemento/docker.otrs
* Perl::Critic
* Perl::Tidy
* remove Support for CGI-PerlEx
* add SQLite backend, might be useful for devel and testing
* add a Selection for MariaDB in installation process, otherwise same as MySQL
* Check the adaptions done in O-Fork
* use `XML::Compile::WSDL`for SOAP

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
