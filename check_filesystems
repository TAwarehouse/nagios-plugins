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
# Check that there is no difference between configured and actual filesystem
# details, in particular:
# - exported filesystems match /etc/exports
# - mounted filesystems match /etc/fstab
#
###############################################################################

use File::stat;
use Getopt::Long;
use Time::Local;
use Socket;

use warnings;
use strict;

# nagios variables

my $CRITICAL = 2;
my $WARNING = 1;
my $OK = 0;

my $test = 'CHECK_FILESYSTEMS';

sub critical { print "$test CRITICAL: @_\n"; exit $CRITICAL }
sub warning { print "$test WARNING: @_\n"; exit $WARNING }
sub ok { print "$test OK: @_\n"; exit $OK }

my @critFailures;
my @warnFailures;

###############################################################################
# Handle options

# No options!

###############################################################################
# Subroutines

# Compare two sets, aka hashes
sub compareSets {
    my $s1 = shift;
    my $s2 = shift;

    my (%t,$k);
    foreach $k (keys %$s1) { $t{$k} += 1 }
    foreach $k (keys %$s2) { $t{$k} += 2 }

    while (my($key,$value) = each(%t)) {
	return 0 if $value != 3;
    }

    return 1;
}

# Expand all symlinks in a path
sub cleanupPath {
    my @right = split('/',$_[0]);
    my @left = shift @right;

    while (@right) {
	push @left,(shift @right);
	my $link = readlink(join('/',@left));
	@left = split('/',cleanupPath($link)) if $link;
    }

    return join('/',@left);
}

# Convert a hostname pattern to an ip address if possible
sub cleanupHostname {
    my $hostPattern = shift;
    my @ipaddr = inet_aton($hostPattern);
    if (defined($ipaddr[0])) {
	return inet_ntoa(@ipaddr);
    } else {
	return $hostPattern;
    }
}

# Read in file, split each line on whitespace
sub readFile {
    my $filename = shift;
    my @retval;
    open F,$filename or critical("Cannot open $filename");
    foreach (<F>) {
	s/#.*//;    # Strip comments
	s/^\s*//;   # Strip leading whitespace
	/^\s*$/ and next;    # Skip blank lines
	push @retval, [ split ];
    }
    close F;
    return @retval;
}

###############################################################################
# Mounted filesystems

# Load in the mounted filesystems from /proc/mounts

my %mountedFs;
my @autoFsRoots;

for my $line (readFile("/proc/mounts")) {
    my ($filesystem,$mountpoint,$type) = @$line;
    if ($type eq 'autofs') {
	push @autoFsRoots,$mountpoint;
	next;
    }
    if ($type eq 'nfs') {
	$mountedFs{$mountpoint} = $filesystem;
	next;
    }
}

# Remove all nfs mounts that are under an automount point
for my $automount (@autoFsRoots) {
    foreach my $fs (keys %mountedFs) {
	if ($fs =~ m%$automount/.*%) {
	    delete $mountedFs{$fs};
	}
    }
}

# Load in the configured filesystems from /proc/mounts

my %configuredMount;

for my $line (readFile("/etc/fstab")) {
    my ($filesystem,$mountpoint,$type) = @$line;
    $configuredMount{$mountpoint} = $filesystem if $type eq 'nfs';
}

# Now compare them.

# Check that the mounted filesystem is configured
while (my($key,$value) = each (%mountedFs)) {
    if (!$configuredMount{$key}) {
	push @critFailures, "Mount missing from /etc/fstab: $key";
	next;
    }
    if ($configuredMount{$key} ne $value) {
	push @critFailures, "Configured mount at $key is different from actual mount";
    }
}

# Check that the configured filesystem is mounted
while (my($key,$value) = each (%configuredMount)) {
    if (!$mountedFs{$key}) {
	push @critFailures, "/etc/fstab entry not mounted: $key";
	next;
    }
}

###############################################################################
# Exported filesystems

# Load in exported filesystems from /etc/exportfs

my %configuredExports;

for my $line (readFile("/etc/exports")) {
    my ($filesystem,$host) = @$line;
    $filesystem = cleanupPath($filesystem);
    ($host) = split(/\(/,$host);
    # Change all hostnames to IP addresses
    $host = cleanupHostname($host);
    $configuredExports{$filesystem}{$host} = 1;
}

# Load in actual exported filesystems from /var/lib/nfs/xtab

my %actualExports;

for my $line (readFile("/var/lib/nfs/etab")) {
    my ($filesystem,$host) = @$line;
    ($host) = split(/\(/,$host);
    # Change all hostnames to short
    $host = cleanupHostname($host);
    $actualExports{$filesystem}{$host} = 1;
}

# Now compare them

# Check that the exported filesystem is configured
while (my($key,$value) = each(%actualExports)) {
    if (!$configuredExports{$key}) {
	push @critFailures,"Export missing from /etc/exports: $key";
	next;
    }
    if (!compareSets($configuredExports{$key},$actualExports{$key})) {
	push @critFailures,"Configured export of $key is different from actual export";
	next;
    }
}

# Check that the configured filesystem is exported
while (my($key,$value) = each(%configuredExports)) {
    if (!$actualExports{$key}) {
	push @critFailures,"/etc/exports entry not exported: $key";
	next;
    }
}

###############################################################################
if (@critFailures) {
    critical(join(';',@critFailures));
} elsif (@warnFailures) {
    warning(join(';',@warnFailures));
} else{
    ok("configured exports and mounts match actual");
}
