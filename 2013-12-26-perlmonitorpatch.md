---
title: Fix Perl with Perl
author: Vincent Engelmann
---


I overheard a colleague complaining that all of our Linux CPU checks were reporting "0%" at seemingly random times. I realized I wasn't quite sure how a Nagios agent would check CPU utilization, so I logged into one of the servers to investigate.

What I found was an old shell script that was executing ``mpstat`` and using ``tail`` and ``nawk`` to grab the cpu metric. The variable that was meant to catch the CPU usage percentage was defined before hand at 0, so when splitting by whitespace in ``nawk`` didn't work, the variable never obtained the CPU percentage from ``mpstat``. I decided to rewrite the shell script in Perl (it was my preferred approach at the time). 

The hard part came when I realized this same faulty shell script was being used across more than 250 servers.

I had remembered there were several SSH modules for Perl. Could I in some way leverage them to update the script across all the servers?

Of course I could!

First I loaded up the new cpu check:

```perl
my $contents = do {
                  local $/ = undef; 
                  open my $fh2, "<", "NEW_check_cpu.txt"
                    or die ": could not open file: $!";
                  <$fh2>;
               };
```

Then I created the command that I would be using for each server where I needed to replace the shell script:

```perl
my $command = "cd /home/username/nrpe/libexec && cp -f check_cpu check_cpu.20130122
    && echo -E \'$contents\' > check_cpu && echo success";
```

After some wrangling, I managed to get Net:SSH2 to connect to the servers and deploy the Perl script:

```perl
foreach my $server ( @serverlist ){
	my $ssh = Net::SSH2->new( trace => -1 );
	if ( !$ssh->connect($server) ) {
		print $logfh "could not connect to $server\n";
		next;
	}
	print $logfh "connected to $server\n";
	$ssh->auth_keyboard( 'someuser', 'somepasswd');
	print $logfh "authenticated on $server\n";
	my $chan = $ssh->channel();
	$chan->exec( $command ); # the magic!
	my $successful = 0;
	while (<$chan>) {
		if ( /success/ ){
			print $logfh "command successful on $server\n";
			$successful = 1;
		}

	}
	if ($successful == 0) {
		print $logfh "command NOT successful on $server\n"
	}
}
```

There is one trick here that initially had me stumped for a while. I needed to expand the ``$command`` variable in the perl script, but I needed to single quote the command line in the shell in order to prevent the shell from trying to execute or expand variables in the replacement script.  But now the backslashes were being interpreted by the shell!

 The answer ended up being the ``-E`` option for ``echo`` (specifically, ``disable interpretation of backslash escapes (default)``). 
