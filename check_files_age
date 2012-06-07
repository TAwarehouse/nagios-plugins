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
# Finds all files that match a pattern and checks the age of the youngest.
#
# Option:
#  -w age in hours for warning
#  -c age in hours for critical
###############################################################################

use Getopt::Long;

use warnings;
use strict;

# nagios variables

my $CRITICAL = 2;
my $WARNING = 1;
my $OK = 0;

my $test = 'FILES_AGE';

###############################################################################
# Handle options

# default options
my $warning = 25;
my $critical = 30;
my $directory;
my $pattern;

GetOptions(
	   "w=i" => \$warning,
	   "c=i" => \$critical,
	   "d=s" => \$directory,
	   "p=s" => \$pattern,
) or die "Cannot parse options, $!\n";

$directory or die "Must specify a directory with -d";
$pattern or die "Must specify a pattern with -p";

###############################################################################
sub mtime() { return (stat($_[0]))[9] }

opendir(DIR,$directory) or die "Cannot open directory $directory, $!\n";
my @files = grep(/$pattern/, readdir(DIR));
closedir(DIR);

my $file = shift @files;
my $youngestAge = &mtime("$directory/$file");
my $youngestFile = $file;

for my $file (@files) {
  my $mtime = &mtime("$directory/$file");
  if ($mtime > $youngestAge) {
    $youngestAge = $mtime;
    $youngestFile = $file;
  }
}

my $age = int((time - $youngestAge)/(60*60));

my $msg = "Youngest file $file is $age hours old";
my $status;

if ($age > $critical) {
  $msg = "$test CRITICAL: $msg";
  $status = $CRITICAL;
} elsif ($age > $warning) {
  $msg = "$test WARNING: $msg";
  $status = $WARNING;
} else {
  $msg = "$test OK: $msg";
  $status = $OK;
}

print $msg,"\n";
exit $status;
