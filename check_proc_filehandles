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
# Checks the number of filehandles for each process
# Option:
#  -w percentage for warning
#  -c percentage for critical
###############################################################################
use Getopt::Long;

use warnings;
use strict;

###############################################################################
# nagios variables

my $CRITICAL = 2;
my $WARNING = 1;
my $OK = 0;

my $test = 'PROC_FH';

###############################################################################
# Handle options

# default options
my $warning = 80;
my $critical = 90;

GetOptions(
	   "w=i" => \$warning,
	   "c=i" => \$critical,
) or die "Cannot parse options, $!\n";

###############################################################################

my $msg = "$test OK: No processes near fd limit\n";
my $status = $OK;

opendir(PROCDIR,'/proc') or die "Cannot open /proc, $!\n";
my @pids = grep /^\d+$/, readdir(PROCDIR);
closedir(PROCDIR);

for my $pid (@pids) {

  # Processes may disappear while we're scanning /proc, so try to handle that case

  # Find the per-process limit from /proc/<pid>/limits
  my $limit = 1024;
  if (open LIMITS, "/proc/$pid/limits") {
    for (<LIMITS>) {
      /Max open files\s+(\d+)/ and $limit = $1;
    }
    close LIMITS;
  }

  # Now find the number of open file descriptors
  my $fdCount = 0;
  if (opendir(FD,"/proc/$pid/fd")) {
    $fdCount = scalar( grep /^\d/, readdir(FD))
  }

  my $percentage = int(($fdCount * 100)/$limit);

  # If we find a critical, exit immediately.  There may be other bad processes
  # but they can't be worse...
  if ($percentage > $critical) {
    print "$test CRITICAL: Process $pid $percentage% open filehandles: $fdCount/$limit\n";
    exit $CRITICAL;
  }

  # If we find a warning, remember, possibly overwriting other warnings.
  if ($percentage > $warning) {
    $msg = "$test WARNING: Process $pid $percentage% open filehandles: $fdCount/$limit\n";
    $status = $WARNING;
  }
}

print $msg;
exit $status;
