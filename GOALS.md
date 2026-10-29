The fork goals and what's I'm trying to fix.

The initial idea behind all this was not to continue abandoned project (it actually was not completely abandoned when I started) but 
to make working (probably not brilliant and not polished) windows driver what will not require yearly license payments (and not to depend of some corporate monkey business at all) and keep the my own cluster running.

I made homelab project initially based on BaxterPad 2019th setup (it proved to be completely wrong idea, but I put too much money and effort to it to replace by something completely different)
he started from this: https://www.reddit.com/r/DataHoarder/comments/8ocjxz/200tb_glusterfs_odroid_hc2_build/ and later switched to https://www.reddit.com/r/helios64/comments/k9y2u4/5_node_200tb_helios64_build_update/ - this was his last (known for me) design, and I can't find his earlier one odroid-based, but the idea I accepted was a mix of his first gluster-based attempt and this later one (also totally wrong, he never learned on his errors ;)

My setup is also a bunch of HC2's (boards not produced anymore, development and support stopped, so I locked onto ubuntu 18 with they hand-made kernel) initially with lizardfs 3.12 from standard ubuntu package.
It's already had some issues, part of which were fixed by OnlyJob debian maintainer, so I actually took his work as a foundation of mine own.

I'm not a opensource and particularily linux fan (both are crap). Most of my everyday tasks are windows-backed and I need reliable and reasonable fast ... ok... not snaily slow at least access to my storage from windows.

I played with they "ganesha plugin", it looked easy target, but found the ganesha nfs itself is completely crappy shit-software. (*)
I tried to buy windows driver from lizardfs.com but they prices were ridiculous. (they tried to sell me "support" I never needed, especially from them)
I looked on samba... the gluster had almost-working samba plugin, and lizardfs had something like api... but writing samba plugin for samba 4.7 (the one shipped with ubuntu18) proved to be hard and silly task (they later completely changed vfs api so new versions need complete rewrite - much simplier, but absolutely different, most of ubuntu18 era code became dead)

So I resorted to most simple way of doing thing - just compile lizardfs code for windows target and add some shim to replace fuse calls with something available on windows.
Most work were already done - one of the teams behind lizardfs made almost-working port to windows, and there are even TWO fuse-api drivers made by enthusiasts - old Dokan/dokany and newer winfsp.

Actually, it worth only bunch of small shim fixes there and here, and relatively small wibecoded shim translating fuse to winfsp api (winfsp had no lowlevel fuse api... until claude did it)

But during both this work and routine maintenance of the cluster, I dived too deep into the code. If I knew that all in advance, I would never started and never used lizardfs at all.

Few facts:
1. the lizardfs developers (even of great-good era!) were exclusively poor in coding and somewhat great in hiding they incompetence. So not only they code had issues, they tried to hide them instead of fixing properly.
    - worst story of course was the 3.13pre data-eater. Which fired exactly because they actually tried to hide they previous mistake without giving any info to users. And made another TWO fatal ones. One was caught, second was unresolved for 5 years.
    - less dangerous for data but not less weird was the bugs in formatting strings (leading to crashes on arm exactly where were something to urgently report!) - quite easily detectable if... some shithead not especially BLOCKED compile-time warnings.
3. lizardfs clusters on the architecture-mixed hardware are broken dead - authors used architecture-variating method for computing hashes, so they will never match!
4. the mfsmetalogger design is completely broken and absolutely unreliable, you should never trust it! (note baxterpad's lastest config has no shadows at all! All his data are backed only on metaloggers. I did the same error, because just never knew what practices are good and what are dangerously bad)

So I'm triyng to fix at least the ones I can in my spare time.

So far: dangerous goals are reported and cluster stopped until some humans come and fix, not tried to be "repaired" behind the scene.
Most platform-broken code found to the day is fixed (some required to change on-wire format, so my work is not 100% compatible with lizardfs - I'm having no choice, my cluster is multiplatform)
The metalogger is the worst. The task still half-finished. Back the metaloggers by at least one shadow until it fully resolved if you value your data.
The metalogger and metarestorer are the tools of the last resort. Don't trust them.

There are also some minor fixes because the old unmaintained project lost it's ability to be compiled on modern systems years ago.

Also the monitoring tool is rewritten the way it may be runned by both python 2 (original code) and 3. The protocol also slightly changed, exposing more information about metadata healf (lessons were learned)

\* - actually, ganesha things were even worse - the client library on which it was build had endless recursion bug, so it would hang on first readlink() - nobody at lizardfs team seems never even tried to simplest tests of what they coded. \[fixed in ng12 if you bothered on that]
