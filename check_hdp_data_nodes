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
# Check hadoop data nodes, using the list of all slaves in the Hadoop config.
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

###############################################################################
package main;

# nagios variables

my $CRITICAL = 2;
my $WARNING = 1;
my $OK = 0;

my $test = 'HADOOP_DATA_NODES';

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

# Run dfsadmin -report |grep 'Datanodes available:' | awk '{print $3}'
my $slaveCount = ($hdp->slaves());

my $cmd = $hdp->root."/bin/hadoop dfsadmin -report";
foreach (`sudo -u hdfs $cmd`) {
  /^Datanodes available: (\d+)/ or next;
  if ($1 == $slaveCount) {
    ok("Slave count matches datanodes available: $slaveCount");
  } else {
    critical("Slave count is $slaveCount, but there are $1 datanodes available");
  }
}

critical("Cannot contact slaves");
