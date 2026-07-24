# What is Devel::Cover ?

See https://metacpan.org/dist/Devel-Cover/view/lib/Devel/Cover/Tutorial.pod and https://metacpan.org/pod/Devel::Cover.

# Using Devel::Cover for Specific Test Scripts, without Tracking the Web Server

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

# Using Devel::Cover for the Complete Test Suite, including Web Server and Daemon

In that case the command
`HARNESS_PERL_SWITCHES=-MDevel::Cover bin/otobo.Console.pl Dev::UnitTest::Run` does not do the whole job.
This is because the web server and the daemon are not traced.

The recommended approach is to use an Docker image which has Devel::Cover already installed. This can be achieved by:

In the otobo sandbox:

- activate devel:profiling in bin/otobo.CheckModules.pl
- regenerate the cpanfile with bin/otobo.CheckModules.pl --generate-cpanfiles
- build the local Docker images with bin/docker/build_docker_images.sh

In the otobo-docker sandbox:

- use the local Docker images by editing .env, uncomment the line with "otobo:local-11.1.x "
- append docker-compose/testing/coverage.yml to COMPOSE_FILE, the sets PERL5OPT

In the container:

- run the test suite, or whatever should be coverage tested
