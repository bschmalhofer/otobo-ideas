# Introduction

## Who am I ?

- Bernhard Schmalhofer from Munich.pm
- First dabbled with Perl in 1990 while studying physics
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

## What is PSGI ?

Interface for running Perl based application in many environments.

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
- 2019: Rother OSS forks OTOBO from _((OTRS)) Community Edition_. Development by Stefan and Sven.
- 2020-01-30: OTOBO 10.0.0 Beta 1
- 2020-07-13: OTOBO 10.0.1 with Docker support: first part of this talk
- 2022-03-02: OTOBO 10.1.1 with PSGI everywhere: second part of this talk

# The Traditional Architecture OTOBO 10.0

- Apache 2.4 with **mpm_prefork** and **mod_perl**
- CGI scripts running under `ModPerl::Registry`
- relational database in the backend
- Template Toolkit in the frontend
- in between a lot of interface modules, not really a framework

## Excursion 1: mod_perl and [ModPerl::Registry](modperl_registry.md)

# OTOBO 10.0 under Docker with PSGI

The initial requirement was that customers have a simple way of running OTOBO. Inspired by <https://hub.docker.com/r/juanluisbaptiste/otrs/>.

## Excursion2: [Docker Support](docker_10_0.md)

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

In OTOBO 10.0 there was direct access to environment variables.

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

Direct reading from STDIN no longer works. In OTOBO 10.0 there were already fallbacks to using the query objects,
so the reads from STDIN could be removed. Take care to consider not only POSTDATA. PUT and PATCH are also relevant.

    # The body supplied by POST, PUT, and PATCH has already been read in. This should be safe
    # as $CGI::POST_MAX has been set as an emergency brake.
    # For Checking the length we can therefor use the actual length.
    my $Content = $ParamObject->GetParam( Param => uc($RequestMethod) . 'DATA' );

## Writing to STDOUT

## Setting headers

Collect the headers in the response object and attach the content later. `Plack::Response::finalize()` does the job.

    @@ -4245,13 +4330,16 @@ sub CustomerHeader {
             $Param{ColorDefinitions} .= "--col$Color:$ColorDefinitions->{ $Color };";
         }
     
    +    $Self->_AddHeadersToResponseObject(
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
     
## Long polling, Comet

Long polling was not relevant for OTOBO. It could be implemented by returning a long running subroutine in the PSGI response. This approach is planned to be used for delivering attachments in OTOBO.

## Encoding

Decoding and encoding is still done incorrectly.

# Yes, we are hiring

- [OTOBO Jobs](https://otobo.de/de/jobs/)
- [OTOBO Community](https://otobo.de/de/community/)

# Sources

- [OTOBO](https://otobo.de/de/community/)
- [OTRS on Wikipedia](https://de.wikipedia.org/wiki/OTRS)
- [OTOBO on Github](https://github.com/RotherOSS/otobo)
- [OTRS under Docker](https://hub.docker.com/r/juanluisbaptiste/otrs/)
- [Fron CGI to PSGI](https://perlmaven.com/from-cgi-to-psgi-and-starman)
- [Porting guidelines](https://github.com/bschmalhofer/otobo-ideas#psgi-stumbling-blocks)

