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
# Check that root, and only root, has a crontab
# This is useful on servers which allow user logins where you want to
# avoid them from running their own scheduled jobs.
#
###############################################################################

use File::stat;
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

# No options!

###############################################################################
# Look at the list of crontabs

my $gotRootCrontab = 0;
my $rootCrontabNotEmpty = 0;
my @otherCrontabs;

opendir(D,"/var/spool/cron") or critical("Cannot open dir /var/spool/cron");

for my $crontab (grep /[^.]/, readdir D) {
    if ($crontab eq 'root') {
	$gotRootCrontab = 1;
	my $sb = stat("/var/spool/cron/root");
	$rootCrontabNotEmpty = $sb->size > 0;
    } else {
	push @otherCrontabs, $crontab;
    }
}

closedir D;

###############################################################################
# Sort out problems

my @critFailures;
my @warnFailures;

push @critFailures, "No root crontab" if !$gotRootCrontab;
push @critFailures, "Root crontab is empty" if $gotRootCrontab and !$rootCrontabNotEmpty;
push @critFailures, "Extra crontabs: ".join(', ',@otherCrontabs) if @otherCrontabs;

###############################################################################
if (@critFailures) {
    critical(join(';',@critFailures));
} elsif (@warnFailures) {
    warning(join(';',@warnFailures));
} else{
    ok("root and only root has a crontab");
}
