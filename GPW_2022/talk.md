# Introduction

## Who am I ?

- Bernhard Schmalhofer from Munich.pm
- First dabbled with Perl in 1990 while studying physics in Regensburg and at the MPE
- Some years in bioinformatics
- Freelancing since 2008
- Became involved with OTOBO in 2020



## What is this talk about ?

- Advertising: Tell people about the success story OTOBO
- Share findings from modernizing an application with a long history



## What is OTOBO ?

- an open source web based ticketing system, where ticketing refers to help desk, not to concerts
- forked from OTRS in 2019 by Rother OSS
- supports ITSM, that is IT Service Management
- see the [demo](https://demo.otobo.org/otobo/index.pl) with login Lena/Lena



## What is [PSGI](https://metacpan.org/pod/PSGI) and [Plack](https://plackperl.org/) ?

- PSGI is an interface for running Perl based application in many environments
- Plack is the reference implementation.



## Meet the Team

Grit, Stefan, and Sven

![Rother OSS people](https://rother-oss.com/wp-content/uploads/2020/05/Header-rother-OSS-experten-fuer-die-otrs-community-edition-1500x630.jpg)


Harry

![Meet Harry](https://otobo.de/wp-content/uploads/2020/07/OTOBO-Login-495x400.png)



## Timeline

- 2001: OTRS started by Martin Edenhofer
- 2003: OTRS 1.0
- 2003 - 2021: ... many OTRS releases and enhancements
- 2004: Stefan Rother become the first employee of the OTRS GmbH
- 2011: Rother OSS founded by Stefan Rother
- 2017: OTRS 6.0 was released, the basis for OTOBO


- 2018: OTRS AG was founded. The open source release was rebranded as _((OTRS)) Community Edition_.
- 2019: Rother OSS forks OTOBO from _((OTRS)) Community Edition_. Development by Stefan and Sven.
- 2020-01-30: OTOBO 10.0.0 Beta 1
- 2020-07-13: OTOBO 10.0.1 with Docker support. First part of this talk
- 2022-03-02: OTOBO 10.1.1 with PSGI everywhere. Second part of this talk



# traditional OTOBO 10.0

- Apache 2.4 with **mpm_prefork** and **mod_perl**
- CGI scripts running under `ModPerl::Registry`
- relational database in the backend
- Template Toolkit in the frontend
- in between a lot of interface modules, not really a framework


## Excursion 1: [ModPerl::Registry](https://metacpan.org/dist/mod_perl/view/docs/api/ModPerl/Registry.pod)

ModPerl::Registry takes a CGI-script

    #!/usr/bin/env perl
    use v5.30;

    use CGI;

    my $cgi = CGI->new;

    print
        $cgi->header(
            -type       => 'text/html',
            -charset    => 'utf-8',
        ),
        '<h1>Hallo 🌍!</h1>';


and turns it into an mod\_perl request handler

    package ModPerl::ROOT::ModPerl::Registry::opt_otobo_bin_cgi_2dbin_hallo_welt_2epl;
    sub handler {local $0 = '/opt/otobo/bin/cgi-bin/hallo_welt.pl';
    #line 1 /opt/otobo/bin/cgi-bin/hallo_welt.pl
    #!/usr/bin/env perl

    use v5.30;

    use CGI;

    my $cgi = CGI->new;
    print
        $cgi->header(
            -type       => 'text/html',
            -charset    => 'utf-8',
        ),
        '<h1>Hallo 🌍!</h1>';
    }


and somewhere the environment variables and STDIN are set up



# OTOBO 10.0 under Docker with PSGI

- initial requirement was that customers have a simple way of running OTOBO
- prior art by <https://hub.docker.com/r/juanluisbaptiste/otrs/>.
- see [Installing using Docker and Docker Compose](https://doc.otobo.org/manual/installation/10.0/en/content/installation-docker.html)



## Here is a quick runthrough.

- Docker support for OTOBO is based on the official [Perl Docker](https://hub.docker.com/_/perl) image.
- The base image is declared in [otobo.web.dockerfile](https://github.com/RotherOSS/otobo/blob/rel-10_0/otobo.web.dockerfile#L10).
- Starting a Docker container runs [entrypoint.sh](https://github.com/RotherOSS/otobo/blob/rel-10_0/bin/docker/entrypoint.sh#L114).
- **plackup** uses [otobo.psgi](https://github.com/RotherOSS/otobo/blob/rel-10_0/bin/psgi-bin/otobo.psgi#L638).
- Orchestration is done with Docker compose.


## Why Plack and not *your favorite framework* ?

- there was no apparent immediate benefit 
- the required changes should be kept to a minimum.


## Why [Gazelle](https://metacpan.org/pod/Gazelle) ?

- advertised as "a Preforked Plack Handler for performance freaks"
- preforking actually is a requirement.


## [CGI::Emulate::PSGI](https://metacpan.org/pod/CGI::Emulate::PSGI)

- just about no code adaptions needed
- add the wrapper in [otobo.psgi](https://github.com/RotherOSS/otobo/blob/rel-10_0/bin/psgi-bin/otobo.psgi#L509)
- then the [venerable CGI.pm](https://github.com/RotherOSS/otobo/blob/rel-10_0/Kernel/System/Web/Request.pm#L93) can kick in.


## a persistent volume is required

- the directory structure of _/opt/otobo_ is a bit intertwined
- OTOBO packages are installed into the same directories as core OTOBO
- even worse, packages may overwrite files in core OTOBO
- cache files are also written in _/opt/otobo_
- solved by using a [Docker volume](https://github.com/RotherOSS/otobo-docker/blob/rel-10_0/docker-compose/otobo-base.yml#L61) 



# OTOBO 10.1 The real thing

- using the Plack App **otobo.psgi** in all scenarios
- Plack::Handler::Apache2 when running under Apache
- wrapper scripts using **Modperl::Registry** as a fallback


## The Query Object

The query parameters are no longer extracted from the environment variables. The PSGI `$Env` is used instead.

    @@ -87,10 +109,22 @@ sub new {
         # max 5 MB posts
         $CGI::POST_MAX = $ConfigObject->Get('WebMaxFileUpload') || 1024 * 1024 * 5;

    -    # The query is usually constructed from the CGI relevant environment variables, e.g. QUERY_STRING,
    -    # and from reading STDIN.
    -    # In specific cases, e.g. test scripts, a already prepared query object can be passed in.
    -    $Self->{Query} = $Param{WebRequest} || CGI->new();
    +    # query object when PSGI env is passed, the recommended usage
    +    if ( $Param{PSGIEnv} ) {
    +        $Self->{Query} = CGI::PSGI->new( $Param{PSGIEnv} );
    +    }
    +
    +    # query object (in case use already existing WebRequest, e. g. fast cgi or classic cgi)
    +    elsif ( $Param{WebRequest} ) {
    +        $Self->{Query} = $Param{WebRequest};
    +    }
    +
    +    # Use an empty CGI object as a fallback.
    +    # This is needed because the ParamObject is sometimes created outside a web context.
    +    # Pass an empty string, in order to avoid that params in %ENV are considered.
    +    else {
    +        $Self->{Query} = CGI->new('');
    +    }

         return $Self;
     }
    @@ -130,9 +164,11 @@ sub GetParam {


## Environment

- in OTOBO 10.0 there was direct access to environment variables

    @@ -106,11 +113,15 @@ sub ProviderProcessRequest {
             );
         }

    +    # The HTTP::REST support works with a request object.
    +    # Just like Kernel::System::Web::InterfaceAgent.
    +    my $ParamObject = $Kernel::OM->Get('Kernel::System::Web::Request');
    +
         my $EncodeObject = $Kernel::OM->Get('Kernel::System::Encode');

         my $Operation;
         my %URIData;
    -    my $RequestURI = $ENV{REQUEST_URI};
    +    my $RequestURI = $ParamObject->RequestURI();
         $RequestURI =~ s{.*Webservice(?:ID)?\/[^\/]+(\/.*)$}{$1}xms;

         # Remove any query parameter from the URL
    @@ -161,7 +172,7 @@ sub ProviderProcessRequest {
             }
         }

    -    my $RequestMethod = $ENV{'REQUEST_METHOD'} || 'GET';
    +    my $RequestMethod = $ParamObject->RequestMethod() || 'GET';
         ROUTE:
         for my $CurrentOperation ( sort keys %{ $Config->{RouteOperationMapping} } ) {

    @@ -223,7 +234,11 @@ sub ProviderProcessRequest {


## Reading from STDIN

- no more reading from STDIN
- in OTOBO 10.0 there were already fallbacks to using the query objects
- so the reads from STDIN could be eliminated
- Do not consider only POSTDATA. PUT and PATCH are also relevant.

    # The body supplied by POST, PUT, and PATCH has already been read in. This should be safe
    # as $CGI::POST_MAX has been set as an emergency brake.
    # For Checking the length we can therefor use the actual length.
    my $Content = $ParamObject->GetParam( Param => uc($RequestMethod) . 'DATA' );


## Writing to STDOUT

- pass back the content.


## Exiting

- throw an exception.


## Setting headers

- collect the headers in the response object
- attach the content later
- finally call `Plack::Response::finalize()`

See:

    @@ -4245,13 +4330,16 @@ sub CustomerHeader {
             $Param{ColorDefinitions} .= "--col$Color:$ColorDefinitions->{ $Color };";
         }

    +    $Self->\_AddHeadersToResponseObject(
    +        ContentDisposition            => $Param{ContentDisposition},
    +        DisableIFrameOriginRestricted => $Param{DisableIFrameOriginRestricted},
    +    );
    +
         # create & return output
    -    $Output .= $Self->Output(
    +    return $Self->Output(
             TemplateFile => "CustomerHeader$Type",
             Data         => \%Param,
         );
    -
    -    return $Output;
     }


## Exiting

- Throw an exception


## Long polling, Comet

- is not relevant for OTOBO
- could be implemented by returning a sub in the PSGI response
- this approach is planned to be used for delivering attachments


## Encoding

- decoding and encoding is still done in a bad way
- ... but it works



# That's it



# And yes, we are hiring

- [OTOBO Jobs](https://otobo.de/de/jobs/)
- [OTOBO Community](https://otobo.de/de/community/)



# Resources

- [OTOBO](https://otobo.de/de/community/)
- [OTRS on Wikipedia](https://de.wikipedia.org/wiki/OTRS)
- [OTOBO on Github](https://github.com/RotherOSS/otobo)
- [OTRS under Docker](https://hub.docker.com/r/juanluisbaptiste/otrs/)
- [Fron CGI to PSGI](https://perlmaven.com/from-cgi-to-psgi-and-starman)
- [Porting guidelines](https://github.com/bschmalhofer/otobo-ideas#psgi-stumbling-blocks)


- [reveal.js](https://revealjs.com/)
- [App::HTTPThis](https://metacpan.org/dist/App-HTTPThis)
- [These slides](https://github.com/bschmalhofer/otobo-ideas/tree/master/GPW_2022)
