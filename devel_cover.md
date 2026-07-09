# What is Devel::Cover ?

See https://metacpan.org/dist/Devel-Cover/view/lib/Devel/Cover/Tutorial.pod and https://metacpan.org/pod/Devel::Cover.

# Using Devel::Cover for test scripts

Within a running web container:

    # Check whether Devel::Cover is installed
    which cover

    # if not installed
    cpanm --local-lib local Devel::Cover
    which cover

    # a clean slate
    cover -delete var/httpd/htdocs/static/cover_db/

    # do some work, the webserver and the daemon are not under coverage scrutiny
    HARNESS_PERL_SWITCHES=-MDevel::Cover=-db,var/httpd/htdocs/static/cover_db prove -v --merge -I . -I Kernel/cpan-lib scripts/test/TemplateGenerator | tee prove.out

    # create a report
    cover var/httpd/htdocs/static/cover_db

Ponder https://localhost:1490/otobo-web/static/cover_db/coverage.html
or https://localhost:1490/otobo-web/static/cover_db/Kernel-System-TemplateGenerator-pm.html

`HARNESS_PERL_SWITCHES=-MDevel::Cover bin/otobo.Console.pl Dev::UnitTest::Run --test Replace.t`does not work.

General scripts: `perl -MDevel::Cover ...`.
Webserver: `plackup -MDevel::Cover ...`  
