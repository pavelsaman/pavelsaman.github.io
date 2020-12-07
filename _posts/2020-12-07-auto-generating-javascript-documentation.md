---
layout: page
title:  "Auto-generating JavaScript Documentation"
date:   2020-12-05 17:30 +0200
categories: other
---

Writing documentation could be a real trouble mostly because one of the two reasons: 1) it takes time, 2) documentation has a tendency not to stay in sync with the actual code. I've recently written some JavaScript code and was in need to convert [JSDoc](https://en.wikipedia.org/wiki/JSDoc) comments into Markdown used in README.md. I've not found any tool I could use here, so I've eventually took some time to write a Perl script to do it for me. Here it is.

First of all, what is JSDoc? JSDoc is a markdown language used to annotate JavaScript code. You can learn about it on Wiki [here](https://en.wikipedia.org/wiki/JSDoc). It's a good idea to use something like this as opposed to just "normal" unstructured comments because:

- it's agreed upon, so there're tools that can auto-generate parts of it
- IDEs know how to parse it, so they can show it nicely rendered for you

An example from VSCode could look like this:

![image](/images/jsdoc.png)

Those comments above the functions are nicely rendered when I hover above the function name.

To auto-generate JSDoc comments, I use [Add jsdoc comments](https://marketplace.visualstudio.com/items?itemName=stevencl.addDocComments) extension in VSCode.

However, how to write README.md? Well, there're many ways to write it, but for my little one-man project, I decided to go with a simple list of functions along with information I write in JSDoc above every function that's part of the API. That basically translates into converting all JSDoc comments into README.md markdown file.

There's perhaps a tool that can already do it. I searched on Google for about 10 minutes and didn't find anything useful. Some posts on Stackofverflow say there are tools for converting JSDoc comments into HTML, and them tools for converting HTML to markdown. That sounds like too much trouble for my couple of functions.

Therefore, I came up with a Perl script that simply takes one command-line option where I specify the folder with my JavaScript files from which I want to take the comments. Let's say that my functions are in `modules/`, then I run my script like this: `$ doc-parser.pl -f ./modules`.

The script itself looks like this:

```perl
#!/usr/bin/perl

use strict;
use warnings;
use Readonly;
use Carp qw(croak);
use Fcntl qw(:DEFAULT);
use Getopt::Long;

our $VERSION = 1.000;

Readonly my $INTO_FILE      => 'README.md';
Readonly my $HEADING        => '###';
Readonly my $LANGUAGE       => 'javascript';
Readonly my $DEFAULT_ENTRY  => q{.};
Readonly my $CURRENT_FOLDER => q{.};
Readonly my $UP_FOLDER      => q{..};

sub TRUE  { return 1; };
sub FALSE { return 0; };

sub is_js_file {
    my $file_name = shift;

    if ($file_name =~ /[.]js$/xms) {
        return TRUE;
    }

    return FALSE;
}

sub is_not_folder {
    my $file_name = shift;

    if (not -d $file_name) {
        return TRUE;
    }

    return FALSE;
}

sub is_comment_line {
    my $line_string = shift;

    if ($line_string =~ /^([\/][*]{2}|[ ][*])/xms) {
        return TRUE;
    }

    return FALSE;
}

sub has_no_comments {
    my $comments_ref = shift or return;

    if (scalar @{ $comments_ref } == 0) {
        return TRUE;
    }

    return FALSE;
}

sub has_comments {
    my $comments_ref = shift;

    return not has_no_comments($comments_ref);
}

sub find_js_files {
    my @queue = @_;
    my @files = ();

    LEVEL:
    while (@queue) {
        my $file_name = shift @queue;

        if (is_not_folder($file_name)) {

            if (is_js_file($file_name)) {
                push @files, $file_name;
            }

            next LEVEL;
        }
        opendir my $dh, $file_name or next LEVEL;

        FOLDER:
        while (my $sub = readdir $dh) {
            next FOLDER if $sub eq $CURRENT_FOLDER or $sub eq $UP_FOLDER;
            push @queue, "$file_name/$sub";
        }

        closedir $dh;
    }

    return @files;
}

sub get_function_name {
    my $full_file_name = shift;

    # files are named after functions inside them, get the function name
    my @file_name_parts = split /\//xms, $full_file_name;
    my $file_name = $file_name_parts[scalar @file_name_parts - 1];
    @file_name_parts = split /[.]/xms, $file_name;

    return $file_name_parts[0];
}

sub process_file_lines {
    my $file_name          = shift;
    my @extracted_comments = ();

    sysopen my $source_file, $file_name, O_RDONLY or return;

    while (my $line = <$source_file>) {
        if (is_comment_line($line)) {
            push @extracted_comments, $line;
        }
    }

    close $source_file;
    return \@extracted_comments;
}

sub extract_comments {
    my @files              = @_;
    my $extracted_comments = {};

    while (@files) {
        my $file_name = shift @files;
        my $function_name = get_function_name($file_name);

        my $comments_ref = process_file_lines($file_name);
        if (has_comments($comments_ref)) {
            $extracted_comments->{$function_name} = $comments_ref;
        }
    }

    return $extracted_comments;
}

sub include_heading {
    my $filehandle    = shift;
    my $function_name = shift;

    printf {$filehandle} "%s %s%s\n\n", $HEADING, $function_name, '()';
    return;
}

sub include_comments {
    my $filehandle   = shift;
    my $comments_ref = shift;

    printf {$filehandle} "%s%s\n", '```', $LANGUAGE;

    foreach my $line (@{ $comments_ref }) {
        printf {$filehandle} '%s', $line;
    }

    printf {$filehandle} "%s\n\n", '```';
    return;
}

sub process_comments {
    my $extracted_comments_ref = shift;

    unlink $INTO_FILE;
    sysopen my $result_file, $INTO_FILE, O_WRONLY|O_CREAT or return;

    FUNCTION:
    foreach my $function (sort keys %{ $extracted_comments_ref }) {
        next FUNCTION if has_no_comments($extracted_comments_ref->{$function});

        include_heading($result_file, $function);
        include_comments($result_file, $extracted_comments_ref->{$function});
    }

    close $result_file;
    return;
}

sub get_entry_folder {

    GetOptions(
        'f|folder=s' => \ my $entry_folder
    );

    if (not $entry_folder) {
        return $DEFAULT_ENTRY;
    }

    return $entry_folder;
}

###############################################################################

sub run {
    my $entry = get_entry_folder();
    my @js_source_files = find_js_files($entry);
    my $extracted_comments = extract_comments(@js_source_files);
    process_comments($extracted_comments);

    return;
}

run();
exit 0;
```

I've included that in the JavaScript repo as well, you can clone it from [there](https://github.com/pavelsaman/useful).

It nicely generates a [README.md](https://github.com/pavelsaman/useful/blob/master/README.md) for me with all the JSDoc comments.

The script itself is not very configurable, I have no further plans with it, so why bother turning it into some fancy Perl module.

I'll just mention that if you already have a README.md file and run this Perl script, it will delete the initial file and create a new one, so it's a little bit descructive. It also assumes there's one commented function per file - it turns a filename into a heading in README.md. This is perhaps something I might need to change later because I guess one can have more commented functions in one file. However, so far this has worked just fine for me.

It's a simple tool for one particular task. But even such scripts are sometimes necessary to create. It does eventually save a lot of time as well as it allows me to stay focused on the project, not on supportive tasks like writing documentation.
