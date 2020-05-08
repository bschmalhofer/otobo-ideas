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

### Link /opt/otobo to Git checkout

How would this work? SetPermissions.pl messes with the whole dir.

### Linking an individual file or dir

otobo> cd /opt/otobo/Kernel/Modules && ln -b -s ~bernhard/devel/OTOBO/otobo/Kernel/Modules/Installer.pm
otobo> symlinks -r /opt/otobo

### Aliases

alias otobo_restart="sudo systemctl restart apache2.service"

## SEE ALSO
 
 * http://otobo.de
 * [RotherOSS Github repositories](https://github.com/RotherOSS/otobo)

