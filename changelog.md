not true changelog but bugs fixed list in no particlular order

most bugs are on armhf architecture (or just the builds without libJudy)

1. printf formatting was bitness-incorrect, so long values sometimes printed as shorts, mostly leading to sigsegvs - exactly when it WAS something urgent to print
2. shadows hangs while attempted to load big metadata changes (restarted after massive filesystem changes for example - rm -rf about million short files on slow arm platform) because used wrong checks of already validated data
3. but not forever - it also has two timers set on startup, chart code to measure own cpu usage, so it will never complete metadata reading - nobody handles those timers during startup phase, so they fired without properly set signal hanler, killing shadow master
4. metadata "crc" checks was platform-depended (worse, used function guarantied by c++ docs to produce same results only on same run within same binary, no warranties even for separate second run)
5. nonfatal but ...er... "interesting" behavior to block signals on startup. so you can't stop the daemon in the middle of long synchronous task, whatever it be - shutdown master before the shadow - it will hang forever trying to reconnect. No ctrl-c, no correct shutdown from systemd, and as usual - no clues in logs/console what happened. It looked just as a daemon completely hangs.

