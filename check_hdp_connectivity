#!/usr/bin/perl
#  Copyright 2012 TripAdvisor, LLC
#
#  Licensed under the Apache License, Version 2.0 (the "License");
#  you may not use this file except in compliance with the License.
#  You may obtain a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
#  Unless required by applicable law or agreed to in writing, software
#  distributed under the License is distributed on an "AS IS" BASIS,
#  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#  See the License for the specific language governing permissions and
#  limitations under the License.
#
# Check hadoop connectivity, using the list of all slaves in the Hadoop config.
#
# Check that the master, this node, can reach all slaves
#
# Options:
# --hadoop_root <dir> : MANDATORY, specify root of all hadoop files
#
###############################################################################

use strict;
use warnings;

use FindBin qw($Bin);
use lib $Bin;

use Getopt::Long;

###############################################################################
package Hadoop;

# sample root dir: /usr/lib/hadoop

sub new {
    my $class = shift;
    my $root = shift;
    my $self = bless {}, $class;
    $self->{root} = $root;
    $self->{confFile} = "$root/conf/slaves";
    $self->readConf();
    return $self;
}

sub readConf {
    my $self = shift;
    open CONF, $self->{confFile} or main::critical("Cannot open $self->{confFile}, $!");
    
    foreach (<CONF>) {
	s/#.*//;   # Strip comments
	s/^\s+//;  # Strip leading whitespace
	s/\s+$//;  # Strip trailing whitespace
	/^$/ and next;   # Skip blank lines

	my ($hostname) = split;
	$self->{conf}{$hostname} = {
	    hostname => $hostname,
	    is_slave => 't',
	    is_master => 'f',
	};	
    }
}

# Return the hadoop root
sub root { return $_[0]->{root} }

# Return a list of slaves
sub slaves {
    my $self = shift;
    return grep { $self->{conf}{$_}{is_slave} eq 't' }  keys %{$self->{conf}};
}

# Run a command on all slaves using slaves.sh, return output as a list with newlines stripped.
# All slaves must allow the "nagios" user  to SSH into them without a password.  So the public
# key of the host this runs on must be in each slave's ~nagios/.ssh/authorized_keys file.
sub runOnSlaves {
    my $self = shift;
    my $cmd = shift;
    my $fullCmd = "HADOOP_HOME=".$self->root." ".$self->root."/bin/slaves.sh $cmd";
    my @output = `$fullCmd`;
    foreach (@output) { chomp }
    return @output;
}

###############################################################################
package Set;

# This package treats a hash reference as a set

# Return items in A and not in B as a set
sub inAnotB  {
    my $class = shift;
    my $a = shift;
    my $b = shift;
    my %c = %$a;
    delete @c{keys %$b};
    return \%c;
}

# Convert to a space-separated list
sub str {
    my $class = shift;
    my $a = shift;
    return join(' ',keys %$a);
}

# Return True if the set is empty
sub isEmpty { return !%{$_[1]} }

# Return True if two sets are identical
sub equal {
    my $class = shift;
    my $a = shift;
    my $b = shift;
    return ( Set->isEmpty(Set->inAnotB($a,$b)) && Set->isEmpty(Set->inAnotB($b,$a)) );
}

###############################################################################
package main;

# nagios variables

my $CRITICAL = 2;
my $WARNING = 1;
my $OK = 0;

my $test = 'HADOOP_CONNECTIVITY';

sub critical { print "$test CRITICAL: @_\n"; exit $CRITICAL }
sub warning { print "$test WARNING: @_\n"; exit $WARNING }
sub ok { print "$test OK: @_\n"; exit $OK }

###############################################################################
# Options

my $hadoop_root;

GetOptions(
    "hadoop_root=s" => \$hadoop_root,
) or main::critical("Cannot parse options, $!");

$hadoop_root or main::critical("Must specify hadoop root with --hadoop_root");

###############################################################################

my $hdp = Hadoop->new($hadoop_root);

# Run 'hostname' on each slave, keep track of those that returned the correct value
my %found = ();
foreach ($hdp->runOnSlaves("hostname")) {
    my ($host,$long) = split(': ');
    if ($long =~ /^$host\..*/) {
	$found{$host} = 1;
    }
}

my %slaves = map { $_ => 1 } $hdp->slaves;

# There are three cases:
# - all slaves responded
ok("All slaves responded") if Set->equal(\%found,\%slaves);

# - some slaves did not respond
my @noResponse = sort keys %{Set->inAnotB(\%slaves,\%found)};
@noResponse and critical("Slaves did not respond: @noResponse");

# - extra slaves responded (!)
my @extraSlaves = sort keys %{Set->inAnotB(\%found,\%slaves)};
@extraSlaves and critical("Unexpected slaves found: @extraSlaves");
