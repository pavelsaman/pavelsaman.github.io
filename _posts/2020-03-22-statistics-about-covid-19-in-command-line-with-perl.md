---
layout: page
title:  "Statistics about COVID-19 in Command Line with Perl"
date:   2020-03-22 13:07:19 +0200
categories: perl
---

As a regular Linux user, I'm aware of the fact that command line can save me a lot of time. In the light of the current COVID-19 pandemic, I'd like to see some statistics printed out in my terminal window. I'll build this utility in Perl, it will consist of two parts - a module for getting the data and a frontend part that will pretty print those data into a terminal window.

### API

A big part of my success in writing this utility is finding a good API where I can get the desired data. That takes a little bit of googling around.

One API I found sufficient is this [wuhan-coronavirus-api](https://ainize.ai/laeyoung/wuhan-coronavirus-api). The data source seems reliable (Johns Hopkins CSSE), it's updated regularly (couple of times a day), and it's pretty easy to use (open API with no need to register anywhere). So, I decided to choose this one, to be more precise, I'll be using `/jhu-edu/latest` endpoint.

### API data

The API returns data in the following format:

```
[
 {
     "provincestate": "",
     "countryregion": "Thailand",
     "lastupdate": "2020-03-21T17:42:00.010Z",
     "location": { "lat": 15, "lng": 101 },
     "countrycode":{ "iso2": "TH", "iso3": "THA"},
     "confirmed": 322,
     "deaths": 1,
     "recovered": 42
 },
 { ... }
]
```

The only problem is that the object doesn't necessarily have to have all these attributes - `provincestate` could be missing, `countrycode` could be missing (e.g. it's missing for the Czechia for some reason), and even the statistics of people affected could be missing. So it's important to take it into account.

The API also allows to use a query parameter `onlyCountries` which is a boolean value (true or false). However, I aggregate the data on my side since I noticed this param too late and I don't want to rebuild my module now.

### Perl

Let's think about what I want from my little utility in the first place:

- I'd like to get statistics of total cases, number of recovered people, number of deaths, and what was the last data update
- I'd like to get statistics for either one country, a few countries, or all countries available
- in the frontend part, I'd like to pretty print it in some sort of table where values are aligned

The main flow could look something like this:

```perl
my $corona = Corona::Statistics->new();

# getting data
$corona->get_all_stats();

# filtering only countries I want
my $selected_countries = $corona->get_country_stats(\@countries);
die "No country found\n" if not @$selected_countries;

# pretty print everything to STDOUT
pretty_print();
```

With that said, let's build the module I call `Corona`.

First the constructor:

```perl
sub new {
    my $class          = shift;   
    my $iso2_filename  = shift;
    my $iso3_filename  = shift; 

    # because ConfigReader::Simple can't interpret '~' in path
    $iso2_filename =~ s/^~/$ENV{HOME}/ if $iso2_filename and $ENV{HOME};
    $iso3_filename =~ s/^~/$ENV{HOME}/ if $iso3_filename and $ENV{HOME};

    return bless { "iso2" => $iso2_filename,
                   "iso3" => $iso3_filename, countries => {}
                 } => $class;
}
```

Then the subroutine for getting data from the endpoint. This block will use module `Net::HTTPS` which you most likely need to install with your package manager or using [CPAN](https://metacpan.org/pod/Net::HTTPS).

```perl
sub get_all_stats {
    my $self = shift;

    # if I already have data, don't call the endpoint again
    return DATA_IS_PRESENT if keys %{ $self->{countries} };  

    # new request    
    my $req = Net::HTTPS->new(Host => BASE_URL)
        or croak __PACKAGE__ . "::get_data(): Cannot connect to the server\n";
    $req->write_request(GET => COUNTRY_STATS, 'Accept' => 'application/json');   
    croak __PACKAGE__ . "::get_data(): Cannot get answer from the server\n"
        if not $req->read_response_headers;

    my $response_body = "";

    # read response body as long as there's some data
    while (1) {
        my $buf;
        my $body = $req->read_entity_body($buf, 1024); 
        croak __PACKAGE__ . "get_data(): Error reading the response\n"
            if not defined $body;  
        last if not $body;
        $response_body .= $buf;
    }

    # parse the response body
    $self->_parse_response_body(\$response_body);

    return SUCCESS;
}
```

The subroutine `_parse_response_body()` is the key, so let's show its content here as well:

```perl
sub _parse_response_body {
    my $self              = shift;
    my $response_body_ref = shift;
    
    # http response is a json    
    my $corona_stats = parse_json($$response_body_ref);    

    # also try to look up country codes in dot files in $HOME    
    my $iso2_file = ConfigReader::Simple->new($self->{iso2});
    my $iso3_file = ConfigReader::Simple->new($self->{iso3});

    foreach my $country (@$corona_stats) {  
        my $countryname = lc $country->{countryregion};  
        $self->{countries}->{$countryname}->{countryname}
            = $country->{countryregion};
        $self->{countries}->{$countryname}->{lastupdate}
            = $country->{lastupdate};        
        $self->{countries}->{$countryname}->{confirmed}
            += $country->{confirmed} 
                if defined $country->{confirmed};
        $self->{countries}->{$countryname}->{recovered}
            += $country->{recovered} 
                if defined $country->{recovered};
        $self->{countries}->{$countryname}->{deaths}
            += $country->{deaths} 
                if defined $country->{deaths};
        
        if (defined $country->{countrycode}->{iso2}) {
            $self->{countries}->{$countryname}->{iso2}
                = lc $country->{countrycode}->{iso2};
        }
        # try to look up country codes in config files
        else {            
            $self->{countries}->{$countryname}->{iso2}
                = lc $iso2_file->get($countryname)
                    if $iso2_file->get($countryname) and $iso2_file;
        }

        if (defined $country->{countrycode}->{iso3}) {
            $self->{countries}->{$countryname}->{iso3}
                = lc $country->{countrycode}->{iso3};
        }  
        # try to look up country codes in config files
        else {
            $self->{countries}->{$countryname}->{iso3}
                = lc $iso3_file->get($countryname)
                    if $iso3_file->get($countryname) and $iso3_file;
        }           
    }        
}
```

As I mentioned before, I need to make sure attributes exist in the objects returned from the API, otherwise I need to go on.

Now I have all the data in a structure that Perl understands, so the next step would be to get only those country statistics the user asked for on the command line. For that, I'll use the `Getopt::Long` module:

```perl
# get command line switches
my $result = GetOptions(
              'help|h'       => \ my $requested_help,
              'country|c=s@' => \ my @selected_countries_by_user
             );
```

I'll also make the list given by the user unique (and lowercase), so no value will be there more times:

```perl
use List::MoreUtils qw(uniq);
@selected_countries_by_user = uniq @selected_countries_by_user;
# make the values lower case
@selected_countries_by_user
    = map { tr/[A-Z]/[a-z]/; $_ } @selected_countries_by_user;
```

Module `List::MoreUtils` will likely have to be installed as well.

Finally, I have everything needed for building the subroutine `get_country_data()`:

```perl
sub get_country_stats {
    my $self                     = shift;
    my $user_given_countries_ref = shift;    

    my @selected_countries = ();

    # if user requested all countries
    if ($user_given_countries_ref->[0] =~ /^all$/i) {
        my @all_countries = keys %{ $self->{countries} };        
        $user_given_countries_ref = \@all_countries;
    }

    foreach my $country (@$user_given_countries_ref) {
        # find by key first
        if ($self->{countries}->{$country}) {
            push @selected_countries, $self->{countries}->{$country};
        }
        # find by iso2 or iso3
        else {
            my $country_by_iso = $self->_get_country_data_by_iso($country);
            if ($country_by_iso) {
                push @selected_countries, $country_by_iso;
            }
        }
    }   

    return \@selected_countries;
}
```

It will return a reference to a list of objects (statistics). I primarily search by country names which is a key in the object, but the user might also specify iso2 or iso3 country names or combine all of these, therefore I need to search by iso2 or iso3 as well:

```perl
sub _get_country_data_by_iso {
    my $self = shift;
    my $iso  = shift;   
    
    foreach my $country (keys %{ $self->{countries} }) {   
        # skip if a country doesn't have iso2 or iso3 attributes     
        next if not $self->{countries}->{$country}->{iso2};
        next if not $self->{countries}->{$country}->{iso3};

        # return a reference to a particular object found by iso2 or iso3
        return $self->{countries}->{$country}
            if $self->{countries}->{$country}->{iso2} eq $iso;        
        return $self->{countries}->{$country}
            if $self->{countries}->{$country}->{iso3} eq $iso;
    }

    return FAILURE;
}
```

That's all, the whole module looks like this:

```perl
package Corona::Statistics;

use strict;
use warnings;
use Net::HTTPS;
use Carp qw(croak);
use JSON::Parse qw(parse_json);
use ConfigReader::Simple qw(get);

our $VERSION = '1.03';

use constant BASE_URL        => "wuhan-coronavirus-api.laeyoung.endpoint.ainize.ai";
use constant COUNTRY_STATS   => "/jhu-edu/latest";
use constant SUCCESS         => 1;
use constant FAILURE         => 0;
use constant DATA_IS_PRESENT => 0;

sub new {
    my $class          = shift;   
    my $iso2_filename  = shift;
    my $iso3_filename  = shift; 

    # because ConfigReader::Simple can't interpret '~' in path
    $iso2_filename =~ s/^~/$ENV{HOME}/ if $iso2_filename and $ENV{HOME};
    $iso3_filename =~ s/^~/$ENV{HOME}/ if $iso3_filename and $ENV{HOME};

    return bless { "iso2" => $iso2_filename,
                   "iso3" => $iso3_filename, countries => {}
                 } => $class;
}

# gets data from the endpoint
# calls _parse_response_body() for parsing of the data
# returns 1 if there's no error
sub get_all_stats {
    my $self = shift;

    # if I already have data, don't call the endpoint again
    return DATA_IS_PRESENT if keys %{ $self->{countries} };  

    # new request    
    my $req = Net::HTTPS->new(Host => BASE_URL)
        or croak __PACKAGE__ . "::get_data(): Cannot connect to the server\n";
    $req->write_request(GET => COUNTRY_STATS, 'Accept' => 'application/json');   
    croak __PACKAGE__ . "::get_data(): Cannot get answer from the server\n"
        if not $req->read_response_headers;

    my $response_body = "";

    # read response body as long as there's some data
    while (1) {
        my $buf;
        my $body = $req->read_entity_body($buf, 1024); 
        croak __PACKAGE__ . "get_data(): Error reading the response\n"
            if not defined $body;  
        last if not $body;
        $response_body .= $buf;
    }

    # parse the response body
    $self->_parse_response_body(\$response_body);

    return SUCCESS;
}

# parses a http body response and fills the current object with data
# confirmed, recovered, deaths, countrycode attributes might be missing in the object
# countryregion is not unique in the object
# [
# {
#     "provincestate":"",
#     "countryregion":"Thailand",
#     "lastupdate":"2020-03-21T17:42:00.010Z",
#     "location":{"lat":15,"lng":101},
#     "countrycode":{"iso2":"TH","iso3":"THA"},
#     "confirmed":322,
#     "deaths":1,
#     "recovered":42
# },
# { ... }
#]
sub _parse_response_body {
    my $self              = shift;
    my $response_body_ref = shift;
    
    # http response is a json    
    my $corona_stats = parse_json($$response_body_ref);    

    # also try to look up country codes in dot files in $HOME    
    my $iso2_file = ConfigReader::Simple->new($self->{iso2});
    my $iso3_file = ConfigReader::Simple->new($self->{iso3});

    foreach my $country (@$corona_stats) {  
        my $countryname = lc $country->{countryregion};  
        $self->{countries}->{$countryname}->{countryname}
            = $country->{countryregion};
        $self->{countries}->{$countryname}->{lastupdate}
            = $country->{lastupdate};        
        $self->{countries}->{$countryname}->{confirmed}
            += $country->{confirmed} 
                if defined $country->{confirmed};
        $self->{countries}->{$countryname}->{recovered}
            += $country->{recovered} 
                if defined $country->{recovered};
        $self->{countries}->{$countryname}->{deaths}
            += $country->{deaths} 
                if defined $country->{deaths};
        
        if (defined $country->{countrycode}->{iso2}) {
            $self->{countries}->{$countryname}->{iso2}
                = lc $country->{countrycode}->{iso2};
        }
        # try to look up country codes in config files
        else {            
            $self->{countries}->{$countryname}->{iso2}
                = lc $iso2_file->get($countryname)
                    if $iso2_file->get($countryname) and $iso2_file;
        }

        if (defined $country->{countrycode}->{iso3}) {
            $self->{countries}->{$countryname}->{iso3}
                = lc $country->{countrycode}->{iso3};
        }  
        # try to look up country codes in config files
        else {
            $self->{countries}->{$countryname}->{iso3}
                = lc $iso3_file->get($countryname)
                    if $iso3_file->get($countryname) and $iso3_file;
        }           
    }        
}

# selects some country data out of all country data returned by the endpoint
# returns a reference to a list of objects
sub get_country_stats {
    my $self                     = shift;
    my $user_given_countries_ref = shift;    

    my @selected_countries = ();

    # if user requested all countries
    if ($user_given_countries_ref->[0] =~ /^all$/i) {
        my @all_countries = keys %{ $self->{countries} };        
        $user_given_countries_ref = \@all_countries;
    }

    foreach my $country (@$user_given_countries_ref) {
        # find by key first
        if ($self->{countries}->{$country}) {
            push @selected_countries, $self->{countries}->{$country};
        }
        # find by iso2 or iso3
        else {
            my $country_by_iso = $self->_get_country_data_by_iso($country);
            if ($country_by_iso) {
                push @selected_countries, $country_by_iso;
            }
        }
    }   

    return \@selected_countries;
}

# user can add iso2 or iso3 arguments on the command line,
# but these are not keys of the hash, so try to look for
# iso2 or iso3 in the objects
# returns a ref to an object or 0 if nothing is found
sub _get_country_data_by_iso {
    my $self = shift;
    my $iso  = shift;   
    
    foreach my $country (keys %{ $self->{countries} }) {   
        # skip if a country doesn't have iso2 or iso3 attributes     
        next if not $self->{countries}->{$country}->{iso2};
        next if not $self->{countries}->{$country}->{iso3};

        # return a reference to a particular object found by iso2 or iso3
        return $self->{countries}->{$country}
            if $self->{countries}->{$country}->{iso2} eq $iso;        
        return $self->{countries}->{$country}
            if $self->{countries}->{$country}->{iso3} eq $iso;
    }

    return FAILURE;
}


1;

__END__

=head1 NAME

Corona::Statistics - get statistics about COVID-19

=head1 SYNOPSIS

    # initialize
    my $corona = Corona::Statistics->new();

    # get all available data about COVID-19 in different countries    
    $corona->get_all_stats();
    # if called again, the endpoint won't be called because you already have
    # data, so if you need a new set of data, create a new object
    $corona->get_all_stats();

    # get data for only those countries you're interested in
    # pass a ref to a list of country names (or iso2 or iso3 abbreviations)
    my $selected_countries
        = $corona->get_country_stats(\@selected_countries_by_user);

    # if the list is empty, nothing has been found
    # otherwise a list of objects is returned:
    # [
    #    {
    #      'iso3' => 'EST',
    #      'recovered' => 1,
    #      'countryname' => 'Estonia',
    #      'lastupdate' => '2020-03-22T10:42:00.003Z',
    #      'iso2' => 'EE',
    #      'confirmed' => 306,
    #      'deaths' => 0
    #    },
    #    { ... }
    # ]
    die "No country found\n" if not @$selected_countries;

=head1 DESCRIPTION

Corona::Statistics module provides data about COVID-19 pandemic for different
 countries.

The following endpoint is used for getting the data:

    https://wuhan-coronavirus-api.laeyoung.endpoint.ainize.ai/jhu-edu/latest

    More about the endpoint could be found here:

        https://ainize.ai/laeyoung/wuhan-coronavirus-api

=head1 DEPENDENCIES

The following Perl modules are required for this Corona module to work:

=over 8

=item Net::HTTPS

For getting data from endpoint in get_all_stats().

=item JSON::Parse

For parsing a http response body in _parse_response_body().

=item ConfigReader::Simple

Reading country codes from a config file.

=item Carp

For more informative error messages.

=back

=head1 AUTHOR

Pavel Saman

=head1 LICENSE

This library is free software; you may redistribute it and/or modify it under the same terms as Perl itself.

=cut
```

You can find it on github [here](https://github.com/pavelsaman/PerlModules/blob/master/Corona/Corona.pm).

Back to the frontend part. So far, I've built command line switches using the `Getopt::Long` module and the part where I'm making the given list of specified countries unique. Let's see the rest:

```perl
#!/usr/bin/env perl

# frontend part for Corona::Statistics module
# modules Getopt::Long, Corona::Statistics, 
# List::MoreUtils, and Term::Table required

use strict;
use warnings;
use Getopt::Long;
use Corona::Statistics 1.03;
use feature 'say';

# transforms a json to a list needed for Term::Table
sub put_country_stats_to_list {
    my $all_country_stats_ref = shift;    

    # build a list from list of jsons
    my @country_stats_list = ();
    foreach my $country (@$all_country_stats_ref) {
        my $country_stats = [
                             $country->{countryname}, $country->{confirmed},
                             $country->{recovered}, $country->{deaths},
                             $country->{lastupdate}
                            ]
                            ;
        push @country_stats_list, $country_stats; 
    }

    return \@country_stats_list;
}

# gets number of columns for the current terminal window
sub get_number_of_columns {
    return undef if $^O =~ /Win32$/;

    open COL, "tput cols|";
    my $number_of_columns_in_terminal = <COL>;
    close COL;
    
    return $number_of_columns_in_terminal;
}

sub print_help {
    print <<'END_MSG';
corona.pl -c|--country Estonia [-c|--country Italy ...]
corona.pl -c|--country all|ALL

Prints COVID-19 statistics for specified countries.
Prints all country statistics if -c|--coutry all is given.

Print this help with -h|--help option.
END_MSG

}

###############################################################################

# get command line switches
my $result = GetOptions(
              'help|h'       => \ my $requested_help,
              'country|c=s@' => \ my @selected_countries_by_user
             );

# only print help
if ($requested_help or not @selected_countries_by_user) {
    print_help();
    exit 0;
}

# corona object
my $corona = Corona::Statistics->new("~/.2country", "~/.3country");

# get all data; it will die inside get_data() if the call is unsuccessful
$corona->get_all_stats();

# make the list of countries unique
use List::MoreUtils qw(uniq);
@selected_countries_by_user = uniq @selected_countries_by_user;
# make the values lower case
@selected_countries_by_user
    = map { tr/[A-Z]/[a-z]/; $_ } @selected_countries_by_user;

# get desired countries
my $selected_countries
    = $corona->get_country_stats(\@selected_countries_by_user);
die "No country found\n" if not @$selected_countries;

# pretty print everything to STDOUT
use Term::Table;
my $corona_stats_table = Term::Table->new(
    max_width      => get_number_of_columns() || 80,
    pad            => 4,
    allow_overflow => 0,
    collapse       => 1,
    header         => ['country', 'total cases',
                       'recovered', 'deaths', 'last updated'],
    rows           => put_country_stats_to_list($selected_countries),
)
;

say $_ for $corona_stats_table->render;

exit 0;
```

For pretty printing, I'm using module `Term::Table` that gets a list reference as data for the table rows. That's a little trouble since my Corona module returns data as a reference to a list of objects. Therefore, I need to transform the data into a desired format for the `Term:Table` module:

```perl
sub put_country_stats_to_list {
    my $all_country_stats_ref = shift;    

    # build a list from list of jsons
    my @country_stats_list = ();
    foreach my $country (@$all_country_stats_ref) {
        my $country_stats = [
                             $country->{countryname}, $country->{confirmed},
                             $country->{recovered}, $country->{deaths},
                             $country->{lastupdate}
                            ]
                            ;
        push @country_stats_list, $country_stats; 
    }

    return \@country_stats_list;
}
```

I'm also making the table wider if it's possible:

```perl
sub get_number_of_columns {
    return undef if $^O =~ /Win32$/;

    open COL, "tput cols|";
    my $number_of_columns_in_terminal = <COL>;
    close COL;
    
    return $number_of_columns_in_terminal;
}
```

and

```perl
my $corona_stats_table = Term::Table->new(
    max_width      => get_number_of_columns() || 80,
    pad            => 4,
    allow_overflow => 0,
    collapse       => 1,
    header         => ['country', 'total cases',
                       'recovered', 'deaths', 'last updated'],
    rows           => put_country_stats_to_list($selected_countries),
)
;
```

And that's basically all.

### How to use it

You can put the script into your `$PATH`, you also need to tell Perl where to find the module, so you likely want to adjust `PERL5LIB` environment variable, which will add items to `@INC`.

After that, you just type:

```bash
$ corona.pl -c Czechia
+---------+-------------+-----------+--------+--------------------------+
| country | total cases | recovered | deaths | last updated             |
+---------+-------------+-----------+--------+--------------------------+
| Czechia | 995         | 6         | 0      | 2020-03-22T13:42:00.004Z |
+---------+-------------+-----------+--------+--------------------------+
```

or you can specify more countries, you can also mix country names, iso2, and iso3:

```bash
$ corona.pl -c czechia -c rus -c IT
+---------+-------------+-----------+--------+--------------------------+
| country | total cases | recovered | deaths | last updated             |
+---------+-------------+-----------+--------+--------------------------+
| Czechia | 995         | 6         | 0      | 2020-03-22T13:42:00.007Z |
| Russia  | 306         | 12        | 1      | 2020-03-22T13:42:00.007Z |
| Italy   | 53578       | 6072      | 4825   | 2020-03-22T13:42:00.007Z |
+---------+-------------+-----------+--------+--------------------------+
```

or you can print a full list of countries:

```bash
$ corona.pl -c all
+----------------------------------+-------------+-----------+--------+--------------------------+
| country                          | total cases | recovered | deaths | last updated             |
+----------------------------------+-------------+-----------+--------+--------------------------+
| Uganda                           | 1           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Congo (Brazzaville)              | 3           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Estonia                          | 306         | 1         | 0      | 2020-03-22T13:42:00.007Z |
| Kazakhstan                       | 53          | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Brunei                           | 83          | 2         | 0      | 2020-03-22T13:42:00.007Z |
| Ireland                          | 785         | 5         | 3      | 2020-03-22T13:42:00.007Z |
| Malaysia                         | 1183        | 114       | 4      | 2020-03-22T13:42:00.007Z |
| Nepal                            | 1           | 1         | 0      | 2020-03-22T13:42:00.007Z |
| Niger                            | 1           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Kenya                            | 7           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Iran                             | 20610       | 7635      | 1556   | 2020-03-22T13:42:00.007Z |
| Philippines                      | 307         | 13        | 19     | 2020-03-22T13:42:00.007Z |
| Bulgaria                         | 163         | 3         | 3      | 2020-03-22T13:42:00.007Z |
| Paraguay                         | 18          | 0         | 1      | 2020-03-22T13:42:00.007Z |
| France                           | 14431       | 12        | 562    | 2020-03-22T13:42:00.007Z |
| Armenia                          | 160         | 1         | 0      | 2020-03-22T13:42:00.007Z |
| Uruguay                          | 110         | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Eritrea                          | 1           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Cote d\'Ivoire                   | 14          | 1         | 0      | 2020-03-22T13:42:00.007Z |
| Benin                            | 2           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Chile                            | 537         | 6         | 0      | 2020-03-22T13:42:00.007Z |
| Senegal                          | 47          | 5         | 0      | 2020-03-22T13:42:00.007Z |
| Maldives                         | 13          | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Norway                           | 2118        | 1         | 7      | 2020-03-22T13:42:00.007Z |
| Netherlands                      | 3640        | 2         | 137    | 2020-03-22T13:42:00.007Z |
| Montenegro                       | 14          | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Mauritius                        | 14          | 0         | 1      | 2020-03-22T13:42:00.007Z |
| Greece                           | 530         | 19        | 13     | 2020-03-22T13:42:00.007Z |
| Holy See                         | 1           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Kuwait                           | 176         | 27        | 0      | 2020-03-22T13:42:00.007Z |
| Mauritania                       | 2           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Honduras                         | 24          | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Andorra                          | 88          | 1         | 0      | 2020-03-22T13:42:00.007Z |
| Latvia                           | 124         | 1         | 0      | 2020-03-22T13:42:00.007Z |
| United Kingdom                   | 5067        | 67        | 234    | 2020-03-22T13:42:00.007Z |
| Costa Rica                       | 117         | 2         | 2      | 2020-03-22T13:42:00.007Z |
| Pakistan                         | 730         | 13        | 3      | 2020-03-22T13:42:00.007Z |
| San Marino                       | 144         | 4         | 20     | 2020-03-22T13:42:00.007Z |
| Saint Vincent and the Grenadines | 1           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Liberia                          | 3           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Cyprus                           | 84          | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Chad                             | 1           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Zimbabwe                         | 3           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Japan                            | 1007        | 232       | 35     | 2020-03-22T13:42:00.007Z |
| Gabon                            | 4           | 0         | 1      | 2020-03-22T13:42:00.007Z |
| Czechia                          | 995         | 6         | 0      | 2020-03-22T13:42:00.007Z |
| Cruise Ship                      | 712         | 325       | 8      | 2020-03-22T13:42:00.007Z |
| Iraq                             | 214         | 51        | 17     | 2020-03-22T13:42:00.007Z |
| China                            | 81305       | 71857     | 3259   | 2020-03-22T13:42:00.007Z |
| Egypt                            | 294         | 41        | 10     | 2020-03-22T13:42:00.007Z |
| Seychelles                       | 7           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Finland                          | 523         | 10        | 1      | 2020-03-22T13:42:00.007Z |
| Lithuania                        | 83          | 1         | 1      | 2020-03-22T13:42:00.007Z |
| Bahrain                          | 305         | 125       | 1      | 2020-03-22T13:42:00.007Z |
| Switzerland                      | 6575        | 15        | 75     | 2020-03-22T13:42:00.007Z |
| Slovenia                         | 383         | 0         | 1      | 2020-03-22T13:42:00.007Z |
| North Macedonia                  | 85          | 1         | 0      | 2020-03-22T13:42:00.007Z |
| Equatorial Guinea                | 6           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Namibia                          | 3           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Zambia                           | 2           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Lebanon                          | 187         | 4         | 4      | 2020-03-22T13:42:00.007Z |
| Venezuela                        | 70          | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Barbados                         | 6           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Serbia                           | 171         | 1         | 1      | 2020-03-22T13:42:00.007Z |
| Sri Lanka                        | 77          | 1         | 0      | 2020-03-22T13:42:00.007Z |
| Belgium                          | 2815        | 263       | 67     | 2020-03-22T13:42:00.007Z |
| Algeria                          | 139         | 32        | 15     | 2020-03-22T13:42:00.007Z |
| Russia                           | 306         | 12        | 1      | 2020-03-22T13:42:00.007Z |
| Romania                          | 367         | 52        | 0      | 2020-03-22T13:42:00.007Z |
| Colombia                         | 196         | 1         | 0      | 2020-03-22T13:42:00.007Z |
| Saint Lucia                      | 2           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Sweden                           | 1763        | 16        | 20     | 2020-03-22T13:42:00.007Z |
| Congo (Kinshasa)                 | 23          | 0         | 1      | 2020-03-22T13:42:00.007Z |
| Luxembourg                       | 670         | 0         | 8      | 2020-03-22T13:42:00.007Z |
| Dominican Republic               | 112         | 0         | 2      | 2020-03-22T13:42:00.007Z |
| Cabo Verde                       | 3           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Canada                           | 1278        | 10        | 19     | 2020-03-22T13:42:00.007Z |
| US                               | 25417       | 0         | 307    | 2020-03-22T13:42:00.007Z |
| Kyrgyzstan                       | 14          | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Central African Republic         | 3           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Trinidad and Tobago              | 49          | 1         | 0      | 2020-03-22T13:42:00.007Z |
| Kosovo                           | 2           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Ghana                            | 19          | 0         | 1      | 2020-03-22T13:42:00.007Z |
| Albania                          | 76          | 2         | 2      | 2020-03-22T13:42:00.007Z |
| Madagascar                       | 3           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Brazil                           | 1021        | 2         | 15     | 2020-03-22T13:42:00.007Z |
| Azerbaijan                       | 53          | 11        | 1      | 2020-03-22T13:42:00.007Z |
| Liechtenstein                    | 37          | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Bhutan                           | 2           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Argentina                        | 158         | 3         | 4      | 2020-03-22T13:42:00.007Z |
| Martinique                       | 32          | 0         | 1      | 2020-03-22T13:42:00.007Z |
| Bosnia and Herzegovina           | 93          | 2         | 1      | 2020-03-22T13:42:00.007Z |
| Hungary                          | 103         | 7         | 4      | 2020-03-22T13:42:00.007Z |
| Bangladesh                       | 25          | 3         | 2      | 2020-03-22T13:42:00.007Z |
| Slovakia                         | 178         | 0         | 1      | 2020-03-22T13:42:00.007Z |
| Mexico                           | 203         | 4         | 2      | 2020-03-22T13:42:00.007Z |
| Panama                           | 200         | 0         | 1      | 2020-03-22T13:42:00.007Z |
| Malta                            | 73          | 2         | 0      | 2020-03-22T13:42:00.007Z |
| Cuba                             | 21          | 0         | 1      | 2020-03-22T13:42:00.007Z |
| Korea, South                     | 8799        | 1540      | 102    | 2020-03-22T13:42:00.007Z |
| Israel                           | 883         | 36        | 1      | 2020-03-22T13:42:00.007Z |
| Tunisia                          | 60          | 0         | 1      | 2020-03-22T13:42:00.007Z |
| Angola                           | 2           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Saudi Arabia                     | 392         | 16        | 0      | 2020-03-22T13:42:00.007Z |
| Taiwan*                          | 153         | 28        | 2      | 2020-03-22T13:42:00.007Z |
| Vietnam                          | 94          | 17        | 0      | 2020-03-22T13:42:00.007Z |
| South Africa                     | 240         | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Turkey                           | 670         | 0         | 9      | 2020-03-22T13:42:00.007Z |
| Portugal                         | 1280        | 5         | 12     | 2020-03-22T13:42:00.007Z |
| Ukraine                          | 47          | 1         | 3      | 2020-03-22T13:42:00.007Z |
| India                            | 330         | 23        | 4      | 2020-03-22T13:42:00.007Z |
| Uzbekistan                       | 43          | 0         | 0      | 2020-03-22T13:42:00.007Z |
| New Zealand                      | 52          | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Cambodia                         | 53          | 1         | 0      | 2020-03-22T13:42:00.007Z |
| Gambia, The                      | 1           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Oman                             | 52          | 12        | 0      | 2020-03-22T13:42:00.007Z |
| East Timor                       | 1           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Haiti                            | 2           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| United Arab Emirates             | 153         | 38        | 2      | 2020-03-22T13:42:00.007Z |
| Morocco                          | 96          | 3         | 3      | 2020-03-22T13:42:00.007Z |
| Ethiopia                         | 9           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Eswatini                         | 1           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Nigeria                          | 22          | 1         | 0      | 2020-03-22T13:42:00.007Z |
| Poland                           | 536         | 1         | 5      | 2020-03-22T13:42:00.007Z |
| Cameroon                         | 27          | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Belarus                          | 76          | 15        | 0      | 2020-03-22T13:42:00.007Z |
| Iceland                          | 473         | 22        | 1      | 2020-03-22T13:42:00.007Z |
| Guatemala                        | 17          | 0         | 1      | 2020-03-22T13:42:00.007Z |
| Burkina Faso                     | 64          | 5         | 2      | 2020-03-22T13:42:00.007Z |
| Guinea                           | 2           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Bahamas, The                     | 4           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Sudan                            | 2           | 0         | 1      | 2020-03-22T13:42:00.007Z |
| Togo                             | 16          | 1         | 0      | 2020-03-22T13:42:00.007Z |
| Qatar                            | 481         | 27        | 0      | 2020-03-22T13:42:00.007Z |
| Nicaragua                        | 2           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Cape Verde                       | 1           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Somalia                          | 1           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Moldova                          | 80          | 1         | 1      | 2020-03-22T13:42:00.007Z |
| Indonesia                        | 450         | 15        | 38     | 2020-03-22T13:42:00.007Z |
| Australia                        | 1071        | 26        | 7      | 2020-03-22T13:42:00.007Z |
| Papua New Guinea                 | 1           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Rwanda                           | 17          | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Spain                            | 25374       | 2125      | 1375   | 2020-03-22T13:42:00.007Z |
| Suriname                         | 4           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Bolivia                          | 19          | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Ecuador                          | 506         | 3         | 7      | 2020-03-22T13:42:00.007Z |
| Singapore                        | 432         | 140       | 2      | 2020-03-22T13:42:00.007Z |
| Guyana                           | 7           | 0         | 1      | 2020-03-22T13:42:00.007Z |
| Tanzania                         | 6           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Peru                             | 318         | 1         | 5      | 2020-03-22T13:42:00.007Z |
| Jamaica                          | 16          | 2         | 1      | 2020-03-22T13:42:00.007Z |
| El Salvador                      | 3           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Germany                          | 22213       | 233       | 84     | 2020-03-22T13:42:00.007Z |
| Thailand                         | 411         | 42        | 1      | 2020-03-22T13:42:00.007Z |
| Italy                            | 53578       | 6072      | 4825   | 2020-03-22T13:42:00.007Z |
| Monaco                           | 11          | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Fiji                             | 1           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Austria                          | 2814        | 9         | 8      | 2020-03-22T13:42:00.007Z |
| Antigua and Barbuda              | 1           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Jordan                           | 85          | 1         | 0      | 2020-03-22T13:42:00.007Z |
| Djibouti                         | 1           | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Croatia                          | 206         | 5         | 1      | 2020-03-22T13:42:00.007Z |
| Denmark                          | 1420        | 1         | 13     | 2020-03-22T13:42:00.007Z |
| Mongolia                         | 10          | 0         | 0      | 2020-03-22T13:42:00.007Z |
| Afghanistan                      | 24          | 1         | 0      | 2020-03-22T13:42:00.007Z |
+----------------------------------+-------------+-----------+--------+--------------------------+
```

You can print help with `-h` or `--help` switch.

The whole module and the frontend part could be found on [github here](https://github.com/pavelsaman/PerlModules).

**EDIT:**

Based on my latest learnings of Perl, I've updated the code, so it should be free of some obvious mistakes I made back then. Some things I updated are:

- better names of variables etc. based [Perl Best Practises](https://en.wikipedia.org/wiki/Perl_Best_Practices) book, chapter 2
- renamed `Corona::Corona` to `Corona::Statistics`, tha former didn't make much sense and it crated more problems with e.g. `$VERSION` variable
- started using `Carp` module and `croak()` function, this is purely because I wanted to try it out, but there's not much changed in terms of functionality, you just get a bit more info in case of problems
- it can read config files with key value pairs of iso2 and iso3 country codes, the config files could be passed into the constructor
- started using `our $VERSION` syntax
. changed `max_width      => get_number_of_columns() // 80` to `max_width      => get_number_of_columns() || 80` because I want it to be 80 even for e.g. value 0 returned (from whatever reason it might happen)
