This project is my attempt to maintain and fix long-standing bugs in my own lizardfs cluster after the original project mizerable death.
I put it here in the hope that it might be useful to someone else but without any promises to made a finished work or something someone can start own clone from.

Please take into account it's single-man project I do in my spare time. So it will not respect any github or opensource in common accepted standards and practices.
The code itself is available only in debian orig packages, because I use neither github nor git itself.

The issues ARE welcome, but, again, take into account I probably will not have any time for it.
The patches are, mostly, not, because I surely have no time to review, oneliners are probably ok.

The pre-built packages are available for Ubuntu 18.04 both amd64 and ahrmhf only - you will find them on releases page.

There are some changes in configuration and behavior, made to close some nasty data-eating bugs in the original project.
I will publish the documentation when ready to.

For now the most important thing you should know: the lizardfs until 3.13-pre has HUGE issue.
The data-killer bug in 3.13 pre-release was actually made because they (silently! no information was published, nothing was documented) tried to fix it, but made it very hastily and never properly tested.
This fork is made from 3.12 release, and BUG IS JUST THERE. But the handling is comlpetely different. 

The bug itself: the authors (actually as many others that time) had choise wrong method for calculating EC codes. It was not completely they error, it comes from the academic researches who brought us the entire technology.
It is reliable for any EC goals (m,n) where n <=2 - so if you use anything equivalent to raid6, you have not touched by this bug.
If you use some bigger n - your data ARE in danger. Attempt to recover the failed chunk will (with some probability) produce WRONG data. No ways to recover. No ways to detect.

So when this fork's master detect such goals in your mfsgoals.cfg - it will abort immediately (and log the case).
The goal checks are _computed_, not just pre-recorded, and if it aborts - your data are very probable in danger.
(the master don't check if any real files have such goal - only the goals defined in your cluster config  are checked)

So if it refuses to start - read the log carefully. In case of the wrong goal, add as a temporary workaround  ALLOW_UNSAFE_EC_GOALS = 1 to mfsmater.cfg and restart.
Your data STILL IN DANGER, until you scan all your files, change they goals to really recoverable ones and wait until all chunks will be rewriten and all checksums recomputed.
After what - remove wrong goal from config and remove override. 
There are NO way to keep dangerous goals in this fork - they are not recoverable and I will not implement them.

| shape family | status |
|---|---|
| m = 1, 2, 3 (any k ≤ 32) | **provably clean** (all submatrices checked exhaustively) |
| m = 4 | **broken from ec(22,4)** (witness: parity rows {0,1,3}, erased data {0,10,21}) |
| m = 5 | **broken from ec(6,5)** (witness: parity rows {0,1,4}, erased data {0,3,5}) |
| m ≥ 6 | **broken from ec(5,m)**; even ec(4,m) for m ≥ 22 |

Broken-ness is monotone in k and m (V depends only on r,j). Concrete disaster example:
ec(6,5), lose data parts {0,3,5} + parity parts {2,3} — exactly 5 losses, which the
goal promises to survive — and the surviving-set matrix is singular.

The same way it handles broken changelogs (which MAY be produced by original lizardfs because it has almost zero safety checks) - there is NO overrides and no ways to continue.
Your metadata is ALREADY LOST. Hope you have a backup, yeah?
