**OTOBO - migrating from CGI to PSGI**

Bernhard Schmalhofer

German Perl/Raku Workshop conference 2022 in Leipzig

Work on OTOBO sponsored by Rother OSS



## Who am I ?

- Bernhard Schmalhofer from Munich.pm
- Started with Perl in 1990s when studying physics
- Some years in bioinformatics in 2000s
- Freelancing since 2008
- Became involved with OTOBO in 2020



## What is this talk about ?

- Marketing: OTOBO is a Perl success story
- Share experiences from modernizing a CGI-based web application



## What is OTOBO ?

- An open source web based ticketing system
- Where ticketing refers to help desk, not to concerts
- Supports IT Service Management, ITSM
- Forked from OTRS in 2019 by Rother OSS
- Demo at <https://demo.otobo.org/otobo/index.pl>. Log in as Lena/Lena.



## What is [PSGI](https://metacpan.org/pod/PSGI) and [Plack](https://plackperl.org/) ?

- PSGI is an interface for running Perl based application in many environments
- Plack is the reference implementation
- OTOBO is based on basic Plack



## Meet the Team

Grit, Stefan, and Sven

![Rother OSS people](https://rother-oss.com/wp-content/uploads/2020/05/Header-rother-OSS-experten-fuer-die-otrs-community-edition-1500x630.jpg)


Harry

![Meet Harry](https://otobo.de/wp-content/uploads/2020/07/OTOBO-Login-495x400.png)



## Timeline

- 2001: OTRS started by Martin Edenhofer
- 2003: OTRS 1.0 released, OTRS GmbH founded
- 2007: OTRS GmbH becomes the OTRS AG
- 2011: Rother OSS GmbH founded by Stefan Rother, first employee of OTRS GmbH
- 2017: OTRS 6.0 was released, the basis for OTOBO
- 2018: Rebranding as _((OTRS)) Community Edition_ and _OTRS_


- 2019: Rother OSS forks OTOBO from _((OTRS)) Community Edition_. Development by Stefan and Sven.
- 2020-01-30: OTOBO 10.0.0 Beta 1
- 2020-07-13: OTOBO 10.0.1 with Docker support. First part of this talk.
- 2022-03-02: OTOBO 10.1.1 with PSGI everywhere. Second part of this talk.



# traditional OTOBO 10.0

- Apache 2.4 with **mpm_prefork** and **mod_perl**
- CGI scripts running under the handler [ModPerl::Registry](https://metacpan.org/dist/mod_perl/view/docs/api/ModPerl/Registry.pod)
- relational database in the backend
- Template Toolkit in the frontend
- in between a lot of interface modules, not really a framework


## Excursion 1

[ModPerl::Registry](https://metacpan.org/dist/mod_perl/view/docs/api/ModPerl/Registry.pod) takes a CGI-script

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


.. and turns it into a subroutine

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


... and somewhere the environment variables and STDIN are set up and the sub is called



# OTOBO 10.0 in Docker

- a simple way of running OTOBO
- with Gazelle running PSGI
- .. and database, daemon, redis, elasticsearch
- prior art by <https://hub.docker.com/r/juanluisbaptiste/otrs/>
- see [Installing using Docker and Docker Compose](https://doc.otobo.org/manual/installation/10.0/en/content/installation-docker.html)



... traditional setup is still supported



## A quick runthrough

- Docker support for OTOBO is based on the official [Perl Docker](https://hub.docker.com/_/perl) image.
- The base image is declared in [otobo.web.dockerfile](https://github.com/RotherOSS/otobo/blob/rel-10_0/otobo.web.dockerfile#L10).
- Starting a Docker container runs [entrypoint.sh](https://github.com/RotherOSS/otobo/blob/rel-10_0/bin/docker/entrypoint.sh#L114).
- **plackup** serves [otobo.psgi](https://github.com/RotherOSS/otobo/blob/rel-10_0/bin/psgi-bin/otobo.psgi#L638).
- Orchestration is done with Docker compose.


## Why Plack and not *your favorite framework* ?

- there was no apparent immediate benefit
- impedance mismatch
- the required changes should be kept to a minimum.


## Why [Gazelle](https://metacpan.org/pod/Gazelle) ?

- no special reason
- advertised as "a Preforked Plack Handler for performance freaks"
- actually, preforking is a requirement.


## [CGI::Emulate::PSGI](https://metacpan.org/pod/CGI::Emulate::PSGI) does the heavy lifting

- just about no code adaptions needed
- add the wrapper in [otobo.psgi](https://github.com/RotherOSS/otobo/blob/rel-10_0/bin/psgi-bin/otobo.psgi#L509)
- then the [venerable CGI.pm](https://github.com/RotherOSS/otobo/blob/rel-10_0/Kernel/System/Web/Request.pm#L93) can kick in.


## a persistent volume is required

- the directory structure of _/opt/otobo_ is a bit intertwined
- OTOBO packages are installed into _/opt/otobo_
- even worse: files may be overwritten
- cache files are also written into the same dir structure
- solved by using a [Docker volume](https://github.com/RotherOSS/otobo-docker/blob/rel-10_0/docker-compose/otobo-base.yml#L61)
- core files are copied from _/opt/otobo_install/otobo_next_



# OTOBO 10.1: The real thing

- using the Plack App **otobo.psgi** in all scenarios
- Plack::Handler::Apache2 when running under Apache
- wrapper scripts using **Modperl::Registry** as a fallback


## Apache config

    # everything is handled by the PSGI app
    <Location />

        # handle all requests, including the static content, with otobo.psgi
        SetHandler          perl-script
        PerlResponseHandler Plack::Handler::Apache2
        PerlSetVar          psgi_app /opt/otobo/bin/psgi-bin/otobo.psgi

        # Require is supported starting with Apache 2.4.
        # No authentication and all requests are allowed.
        Require all granted

    </Location>


## CGI-scrips as the fallback

    # otobo.psgi looks primarily in $ENV{PATH_INFO}
    local $ENV{PATH_INFO}   = join '/', grep { defined $_ && $_ ne '' } @ENV{qw(SCRIPT_NAME PATH_INFO)};
    local $ENV{SCRIPT_NAME} = '';

    my $CgiBinDir = dirname(__FILE__);
    state $App = Plack::Util::load_psgi("$CgiBinDir/../psgi-bin/otobo.psgi");
    Plack::Handler::CGI->new()->run($App);



# Things to consider



## The Query Object

- query parameters are no longer extracted from `%ENV`
- the PSGI environment `$Env` is used instead

Kernel/System/Web/Request.pm:

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

eradicate direct access to environment variables

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
- could be removed because there were already fallbacks to using the query object
- Do not consider only POSTDATA. PUT and PATCH are also relevant.

Kernel/GenericInterface/Transport/HTTP/REST.pm:

    # The body supplied by POST, PUT, and PATCH has already been read in. This should be safe
    # as $CGI::POST_MAX has been set as an emergency brake.
    # For Checking the length we can therefor use the actual length.
    my $Content = $ParamObject->GetParam( Param => uc($RequestMethod) . 'DATA' );


## Writing to STDOUT

- pass back the complete content
- actually not a big change in OTOBO


## Long polling, Comet

- is not relevant for OTOBO
- could be implemented by returning a sub in the PSGI response
- this approach is planned to be used for delivering attachments


## Setting headers

- collect the headers in the response object
- attach the content later
- finally call `Plack::Response::finalize()`

Kernel/Output/HTML/Layout.pm:

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

Throw an exception

    # The OTOBO response object already has the HTPP headers.
    # Enhance it with the HTTP status code and the content.
    my $ErrorResponse = Plack::Response->new(
        200,
        $Kernel::OM->Get('Kernel::System::Web::Response')->Headers(),
        $Output
    );

    # The exception is caught be Plack::Middleware::HTTPExceptions
    die Kernel::System::Web::Exception->new(
        PlackResponse => $ErrorResponse
    );


... and catch it in a middleware.

    # we might catch an instance of Kernel::System::Web::Exception
    enable 'Plack::Middleware::HTTPExceptions';


## Global variables per Request

- (too) heavily used by OTOBO
- Event handling depends on the destruction of objects at the end of a request
- yet another middleware

otobo.psgi:

    my $ManageObjectsMiddleware = sub {
        my $App = shift;

        return sub {
            my $Env = shift;

            # make sure that the managed objects will be recreated for the current request
            local $Kernel::OM = Kernel::System::ObjectManager->new();

            return $App->($Env);
        };
    };


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
- [OTOBO Design](https://www.sanmiguel.io/de/projekte/ux-ui-muenchen/otobo-userinterface-design)
- [Znuny under Docker](https://hub.docker.com/r/juanluisbaptiste/otrs/)
- [From CGI to PSGI](https://perlmaven.com/from-cgi-to-psgi-and-starman)
- [Porting guidelines](https://github.com/bschmalhofer/otobo-ideas#psgi-stumbling-blocks)


- Presentation with [reveal.js](https://revealjs.com/) and [App::HTTPThis](https://metacpan.org/dist/App-HTTPThis)
- These slides are available at <https://github.com/bschmalhofer/otobo-ideas/tree/master/GPW_2022>
