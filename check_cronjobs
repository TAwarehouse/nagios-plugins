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
# Look for long-running crontab jobs
#
# Options:
# -c <value> threshold for critical in hours, default 48
# -w <value> threshold for warning in hours, default 24
# -x <command-name>:<warning-value>:<critical-value> exception values for particular command.
#
# For crontab jobs, if the command is not specified as an exception value with -x,
# report if the elapsed time is over the warning or critical thresholds; if the
# job is an exception then use the specific values for that command.
#
# -x can be specified multiple times.
#
###############################################################################

use Getopt::Long;
use Time::Local;

use warnings;
use strict;

# nagios variables

my $CRITICAL = 2;
my $WARNING = 1;
my $OK = 0;

my $test = 'CHECK_CRONTABS';

sub critical { print "$test CRITICAL: @_\n"; exit $CRITICAL }
sub warning { print "$test WARNING: @_\n"; exit $WARNING }
sub ok { print "$test OK: @_\n"; exit $OK }

###############################################################################
# Handle options

# default options

my $critical = 48;
my $warning = 24;
my @rawExceptions;

GetOptions(
	   "c=s" => \$critical,
	   "w=s" => \$warning,
	   "x=s" => \@rawExceptions,
) or die "Cannot parse options, $!\n";

# Parse exceptions into exceptions: pattern -> [warning,critical]
# Also keep an ordered list of exceptions so we can go through them in an
# predictable order.
my %exceptions;
my @exceptions;
for my $rawException (@rawExceptions) {
    my ($pattern,$warn,$crit) = split(/:/,$rawException);
    $exceptions{$pattern} = [$warn,$crit];
    push @exceptions,$pattern;
}

###############################################################################
# Get a list of crond processes, times, and their immediate children
my @crondPids;  # List of processes with command 'crond' and ppid != 1
my %cpid;       # ppid -> pid list
my %comm;       # pid -> comm
my %etime;      # pid -> etime
my $cmd = "/bin/ps axwo 'pid ppid etime args'";
foreach (`$cmd`) {
    chomp;
    s/^\s+//;      # Strip leading blanks
    my ($pid,$ppid,$etime,$comm) = split(/\s+/,$_,4);
    push @{$cpid{$ppid}}, $pid;
    $comm{$pid} = $comm;
    $etime{$pid} = $etime;
    push @crondPids, $pid if ($comm eq 'crond' && $ppid != 1);
}

###############################################################################
# Go through all crond processes.  See if any child process matches any
# exception values; if so, use the specific warn/crit values; otherwise
# use the default values.  Collect a list of critical and warning failures.

my @critFailures;
my @warnFailures;

foreach my $pid (@crondPids) {
    # Default values for the check
    my $localCritical = $critical;
    my $localWarning = $warning;
    my $localPattern = 'job';

    # Look through the children processes for a match against an exception pattern
  PATTERN:
    for my $pattern (@exceptions) {
	for my $pid (@{$cpid{$pid}}) {
	    if ($comm{$pid} =~ $pattern) {
		# If a match is found, override the defaults
		($localWarning,$localCritical) = @{$exceptions{$pattern}};
		$localPattern = $pattern;
		last PATTERN;
	    }
	}
    }

    # Find the elapsed time - it has one of the formats DD-HH:MM:SS, HH:MM:SS, MM:SS or SS
    # We only care about the first two since the minimum time we care about is 1 hour.
    my $days = 0;
    my $hours = 0;
    if ($etime{$pid} =~ /(\d+)-(\d+):(\d+):(\d+)/) {
	$days = $1;
	$hours = $2;
    } elsif ($etime{$pid} =~ /(\d+):(\d+):(\d+)/) {
	$hours = $1;
    }
    
    my $etime = $days*24 + $hours;

    # And finally see if the elapsed time is over any threshold
    if ($etime >= $localCritical) {
	push @critFailures, "$localPattern $etime hrs";
    } elsif ($etime >= $localWarning) {
	push @warnFailures, "$localPattern $etime hrs";
    }
}

###############################################################################
if (@critFailures) {
    critical(join(';',@critFailures));
} elsif (@warnFailures) {
    warning(join(';',@warnFailures));
} else{
    ok("No long-running crontab jobs");
}
