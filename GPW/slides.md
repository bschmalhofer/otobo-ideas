# Introduction

## Who am I ?

- Bernhard Schmalhofer Munich.pm
- First dabbled with Perl in 1990 while studying physics
- Freelancing since 2008
- Became involved with OTOBO in 2020

## What is this talk about ?

- Tell people about a success story OTOBO
- Share findings from modernizing an application with a long history

## What is OTOBO ?

- an open source web based ticketing system
- used mostly for service desks
- supports ITSM
- forked from OTRS by Rother OSS

![Rother OSS people](https://rother-oss.com/wp-content/uploads/2020/05/Header-rother-OSS-experten-fuer-die-otrs-community-edition-1500x630.jpg)
![Meet Harry](https://otobo.de/wp-content/uploads/2020/07/OTOBO-Login-495x400.png)


### Timeline

- 2001: OTRS started by Martin Edenhofer
- 2003: OTRS 1.0
- 2004 Stefan Rother become the first employee of the OTRS GmbH
- 2011: Rother OSS founded by Stefan Rother
- 2017: OTRS 6.0
- 2018: ((OTRS)) Community Edition
- 2019: OTOBO forked by Rother OSS: Stefan Rother and Sven Oesterling
- 2020-01-30: OTOBO 10.0.0 Beta 1
- 2020-07-13: OTOBO 10.0.1 
  - support for Docker
  - Docker-based installations: Plack application running under Gazelle
  - non-Docker: Modperl::Registry under Apache 2.4
- 2022-03-02: OTOBO 10.1.1
  - using otobo.psgi in all scenarios
  - Gazelle in the Docker case
  - Plack::Handler::Apache2 when running under Apache
  - Modperl::Registry as a fallback

# old architecture OTOBO 10.0.x

- CGI scripts running under Modperl::Registry
- Database and Template Toolkit

## Recap mod_perl and Modperl::Registry

# OTOBO 10.0 PSGI

## Excursion Docker

- Perl image
- Carton
- plackup

## Excursion Why Plack and not *your favorite framework*

## Excursion Gazelle

## Plack::Handler::CGI

# OTOBO 10.1 The real thing

## The request

## Environment

## exit

## STDIN

## Traps and special cases

# Sources

- [OTOBO](https://otobo.de/de/community/)
- [OTRS on Wikipedia](https://de.wikipedia.org/wiki/OTRS)
