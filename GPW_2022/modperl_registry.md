`ModPerl::Registry` takes a CGI-script

    #!/usr/bin/env perl
    
    use v5.30;
    
    # modules from CPAN
    use CGI;
    
    my $cgi = CGI->new;
    
    print
        $cgi->header(
            -type       => 'text/html',
            -charset    => 'utf-8',
        ),
        '<h1>Hallo 🌍!</h1>';

and turns it into an mod_perl request handler.


    package ModPerl::ROOT::ModPerl::Registry::opt_otobo_bin_cgi_2dbin_hallo_welt_2epl;sub handler {local $0 = '/opt/otobo/bin/cgi-bin/hallo_welt.pl';
    #line 1 /opt/otobo/bin/cgi-bin/hallo_welt.pl
    #!/usr/bin/env perl

    use v5.30;

    # modules from CPAN
    use CGI;

    my $cgi = CGI->new;

    print
        $cgi->header(
            -type       => 'text/html',
            -charset    => 'utf-8',
        ),
        '<h1>Hallo 🌍!</h1>';
    }
