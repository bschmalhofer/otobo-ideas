# Introduction

## Who am I ?

- Bernhard Schmalhofer from Munich.pm
- First dabbled with Perl in 1990 while studying physics
- Freelancing since 2008
- Became involved with OTOBO in 2020

## What is this talk about ?

- Advertising: Tell people about the success story OTOBO
- Share findings from modernizing an application with a long history

## What is OTOBO ?

- an open source web based ticketing system. help desk not concerts
- forked from OTRS in 2019 by Rother OSS
- supports ITSM, that is IT Service Management

## Meet the Team

![Rother OSS people](https://rother-oss.com/wp-content/uploads/2020/05/Header-rother-OSS-experten-fuer-die-otrs-community-edition-1500x630.jpg)
![Meet Harry](https://otobo.de/wp-content/uploads/2020/07/OTOBO-Login-495x400.png)


## Timeline

- 2001: OTRS started by Martin Edenhofer
- 2003: OTRS 1.0
- 2004 Stefan Rother become the first employee of the OTRS GmbH
- ... many releases and enhancement
- 2011: Rother OSS founded by Stefan Rother
- 2017: OTRS 6.0
- 2018: OTRS AG rebrands the open source release as _((OTRS)) Community Edition_
- 2019: Rother OSS forks OTOBO from the community edition
- 2020-01-30: OTOBO 10.0.0 Beta 1
- 2020-07-13: OTOBO 10.0.1 with Docker support
  - support for Docker
  - Docker-based installations: Plack application running under Gazelle
  - non-Docker: Modperl::Registry under Apache 2.4
- 2022-03-02: OTOBO 10.1.1
 

# traditional architecture OTOBO 10.0.x

- Apache 2.4
- CGI scripts running under Modperl::Registry
- Database and Template Toolkit

## Excursion 1: mod_perl and [ModPerl::Registry](modperl_registry.md)

# OTOBO 10.0 under Docker with PSGI

## Excursion Docker

- Perl image
- Carton
- plackup

## Excursion Why Plack and not *your favorite framework*

## Excursion Gazelle

## Plack::Handler::CGI

# OTOBO 10.1 The real thing

- using the Plack App **otobo.psgi** in all scenarios
- Plack::Handler::Apache2 when running under Apache
- wrapper scripts using **Modperl::Registry** as a fallback

## The request

## Environment

## exit

## STDIN

## Traps and special cases

# Yes, we are hiring

[OTOBO Jobs](https://otobo.de/de/jobs/)
[Contribute](https://otobo.de/de/community/)

# Sources

- [OTOBO](https://otobo.de/de/community/)
- [OTOBO on Github](https://github.com/RotherOSS/otobo)
- [OTRS on Wikipedia](https://de.wikipedia.org/wiki/OTRS)
