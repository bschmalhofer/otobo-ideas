# Docker Support in OTOBO 10.0.x

Here is a quick runthrough.

Docker support for OTOBO is based on the official [Perl Docker](https://hub.docker.com/_/perl) image. The base image is declared in [otobo.web.dockerfile](https://github.com/RotherOSS/otobo/blob/rel-10_0/otobo.web.dockerfile#L10).
Starting a Docker container runs [entrypoint.sh](https://github.com/RotherOSS/otobo/blob/rel-10_0/bin/docker/entrypoint.sh#L114).
**plackup** uses [otobo.psgi](https://github.com/RotherOSS/otobo/blob/rel-10_0/bin/psgi-bin/otobo.psgi#L638).

Orchestration is done with Docker compose.

## Why Plack and not *your favorite framework* ?

Because there was no apparent immediate benefit and he required changes should be kept to a minimum.

## Why [Gazelle](https://metacpan.org/pod/Gazelle) ?

Because it is advertised as "a Preforked Plack Handler for performance freaks". The preforked actually is a requirement.

## [CGI::Emulate::PSGI](https://metacpan.org/pod/CGI::Emulate::PSGI)

Just about no code adaptions needed. Add the wrapper in [otobo.psgi](https://github.com/RotherOSS/otobo/blob/rel-10_0/bin/psgi-bin/otobo.psgi#L509). Then the [venerable CGI.pm](https://github.com/RotherOSS/otobo/blob/rel-10_0/Kernel/System/Web/Request.pm#L93) can kick in.

## _/opt/otobo_ must be persistent

Because the directory structure is a bit intertwinded. OTOBO packages are installed into the same directories as core OTOBO. Even worse, packages may overwrite files in core OTOBO.
This is a bit unfortunate. Solved by using a [Docker volume](https://github.com/RotherOSS/otobo-docker/blob/rel-10_0/docker-compose/otobo-base.yml#L61) 
