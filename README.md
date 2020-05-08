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
* Check the requirments

### Other ideas

* Check the installation instructions
* cpanfile
* PSGI
* Starman
* Docker image, see https://github.com/complemento/docker.otrs
* Perl::Critic
* Perl::Tidy
* Devel::NYTProf
* Intellij
* remove Support for FastCGI
* Features to add
  * SQLite backend, might be useful for devel and testing
  * Explicit support for MariaDB
* Check the adaptions done in O-Fork

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

### Aliases

alias otobo_restart="sudo systemctl restart apache2.service"

## SEE ALSO
 
 * http://otobo.de
 * [RotherOSS Github repositories](https://github.com/RotherOSS/otobo)

