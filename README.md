Warplife
========

http://www.kuro5hin.org/story/2013/12/15/05559/463


[+] [P]
Super Secret Warp Life source dump

By cockskin horsesuit [Watch this Diary] in cockskin horsesuit's Diary
Sun Dec 15, 2013 at 12:55:59 AM EST 
Tags: sodomy, horsecock, G4C, Warp Life (all tags)	
	
In honor of the Global Day of Coderetreat, I am posting the full source code to the beta version of Warp Life below.


	Alter M and N for a different grid size.
Important note: Requires roughly 9*M*N file descriptors (ulimit -n) and M*N processes (ulimit -u).  This will probably require adjusting your ulimit and/or the limits in /etc/security/limits.conf and/or running as root (pick any 2).  Or you can shrink the board down really small to use less resources.
Crawford is working hard on improvements to the algorithm to consume less electricity.

To compile: cc -o warplife warplife.c
