# Autoperf (w/ HTTPerf.rb)

![Gem Version](https://badge.fury.io/rb/autoperf.png) &nbsp; ![Build Status](https://travis-ci.org/jmervine/autoperf.png?branch=master)

Autoperf is a ruby driver for [httperf](http://mervine.net/httperf), designed to help you automate load and performance
testing of any web application - for a single end point, or through log replay.

This has been refactored from the original -- [github.com/igrigorik/autoperf](https://github.com/igrigorik/autoperf)
-- to include [HTTPerf.rb](http://mervine.net/gems/httperfrb) as a simplification and to include additional features.
This has been refactored and released with permission from [Ilya Grigorik](http://www.igvita.com/), the origional
author.

More info on the original Autoperf: [http://www.igvita.com/2008/09/30/load-testing-with-log-replay/](http://www.igvita.com/2008/09/30/load-testing-with-log-replay/).

### Requirements

To get started, first download & install httperf: [http://www.hpl.hp.com/research/linux/httperf/](http://www.hpl.hp.com/research/linux/httperf/).

Or for HTTPerf.rb percentile support, install this fork: [http://github.com/jmervine/httperf](http://github.com/jmervine/httperf).

#### Rubies

It's important to note that tests pass for this on the following [Ruby](http://mervine.net/ruby) versions, but I've only used this at length on 1.9.2p290.

* 1.9.3p392
* 2.0.0p0

> Note:
> I was also able to get tests to pass on 1.8.7 and 1.9.2, but not on Traivs and I would consider it unstable on both.


### Installation

    :::shell
    gem install bundler
    echo 'gem "autoperf"' >> Gemfile
    bundle install

Without bundler:

    :::shell
    gem install autoperf

From source:

    :::shell
    git clone git://github.com/rubyops/autoperf.git
    cd autoperf
    gem build autoperf.gemspec
    gem install ./autoperf-VERSION.gem

### Usage

Next, either run a simple test straight from the command line (man httperf), or create
an execution plan for autoperf. If you want to replay an access log from your production
environment, follow these steps:

    :::shell
    # grab last 10000 lines from nginx log, and extract a certain pattern (if needed)
    tail -n 10000 nginx.log | bundle exec make_replay_log > replay_log

Next, configure your execution plan (see [sample.yml](https://github.com/rubyops/autoperf/blob/master/config/sample.yml)), and run autoperf:

    :::shell
    bundle exec autoperf -c sample.yml


Sample output:

    +-------------------------------------------------------------------------------------------------------------------------------+
    | rate | connection_rate_per_sec | request_rate_per_sec | connection_time_avg | errors_total | reply_status_5xx | net_io_kb_sec |
    +-------------------------------------------------------------------------------------------------------------------------------+
    |    5 | 154.4                   | 154.4                | 6.5                 | 0            | 0                | 1176.9        |
    |   10 | 153.4                   | 153.4                | 6.5                 | 0            | 0                | 1169.5        |
    |   15 | 153.0                   | 153.0                | 6.5                 | 0            | 0                | 1166.6        |
    |   20 | 147.4                   | 147.4                | 6.8                 | 0            | 0                | 1123.4        |
    +-------------------------------------------------------------------------------------------------------------------------------+

If your server uses caching, making it pointless to run the same requests over
and over, you can use different requests for each run.

    :::shell
    # Create 10 1000-line files (xa, xb, xc etc)
    split -a 1 requests_path

    # Convert to null-terminated strings
    for x in x?; do tr "\n" "\0" < $x > $x.nul; done

    # run as before, but use the `wlog` line instead of `httperf_wlog` in the conf file
    autoperf -c sample.yml

