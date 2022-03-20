# Docker Support in OTOBO 10.0.x

Docker support for OTOBO is based on the official Perl Docker image. Declared in [otobo.web.dockerfile](https://github.com/RotherOSS/otobo/blob/rel-10_0/otobo.web.dockerfile)
Starting a Docker container runs [entrypoint.sh](https://github.com/RotherOSS/otobo/blob/rel-10_0/bin/docker/entrypoint.sh).
**plackup** uses [otobo.psgi](https://github.com/RotherOSS/otobo/blob/rel-10_0/bin/psgi-bin/otobo.psgi)

## Why Plack and not *your favorite framework* ?

Keep the required changes to a minimum.

## Excursion Gazelle

Does not matter at all, as long as it is preforked. Somebody wrote that Gazelle was the fastest web server.

## CGI::Emulate::PSGI

Just about no code adaptions needed.

## _/opt/otobo_ must be persistent

This is a bit unfortunate. Solved by using a Docker [volume](https://github.com/RotherOSS/otobo-docker/blob/rel-10_0/docker-compose/otobo-base.yml) 
