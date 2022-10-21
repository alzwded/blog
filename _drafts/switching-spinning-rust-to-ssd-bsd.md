```
~> doas fdisk sd0
Disk: sd0       geometry: 19457/255/63 [312581808 Sectors]
Offset: 0       Signature: 0xAA55
            Starting         Ending         LBA Info:
 #: id      C   H   S -      C   H   S [       start:        size ]
-------------------------------------------------------------------------------
 0: 00      0   0   0 -      0   0   0 [           0:           0 ] unused      
 1: 00      0   0   0 -      0   0   0 [           0:           0 ] unused      
 2: 00      0   0   0 -      0   0   0 [           0:           0 ] unused      
*3: A6      0   1   2 -  19457  80  63 [          64:   312581744 ] OpenBSD
```


```
~> doas fdisk sd2
Disk: sd2       geometry: 14593/255/63 [234441647 Sectors]
Offset: 0       Signature: 0xAA55
            Starting         Ending         LBA Info:
 #: id      C   H   S -      C   H   S [       start:        size ]
-------------------------------------------------------------------------------
 0: 00      0   0   0 -      0   0   0 [           0:           0 ] unused      
 1: 00      0   0   0 -      0   0   0 [           0:           0 ] unused      
 2: 00      0   0   0 -      0   0   0 [           0:           0 ] unused      
*3: A6      0   1   2 -  14593  80  62 [          64:   234441583 ] OpenBSD  
```


```
doas fdisk -i sd2
y
```






```
disklabel sd0
# /dev/rsd0c:
type: SCSI
disk: SCSI disk
label: Hitachi HTS54501
duid: db0061b2fe09e6a0
flags:
bytes/sector: 512
sectors/track: 63
tracks/cylinder: 255
sectors/cylinder: 16065
cylinders: 19457
total sectors: 312581808
boundstart: 64
boundend: 312581808
drivedata: 0 
16 partitions:
#                size           offset  fstype [fsize bsize   cpg]
  a:          2097152               64  4.2BSD   2048 16384 12960 # /
  b:          2599656          2097216    swap                    # none
  c:        312581808                0  unused
  d:          8388576          4696896  4.2BSD   2048 16384 12960 # /tmp
  e:         12539328         13085472  4.2BSD   2048 16384 12960 # /var
  f:         12582912         25624800  4.2BSD   2048 16384 12960 # /usr
  g:          2097152         38207712  4.2BSD   2048 16384 12960 # /usr/X11R6
  h:         41943040         40304864  4.2BSD   2048 16384 12960 # /usr/local
  i:          6291456         82247904  4.2BSD   2048 16384 12960 # /usr/src
  j:         12582912         88539360  4.2BSD   2048 16384 12960 # /usr/obj
  k:        211459520        101122272  4.2BSD   2048 16384 12960 # /home
```


```
disklabel sd2
16 partitions:
#                size           offset  fstype [fsize bsize   cpg]
  c:        234441647                0  unused 

df -h
Filesystem     Size    Used   Avail Capacity  Mounted on
/dev/sd0a      986M    515M    422M    55%    /
/dev/sd0k     97.7G   13.6G   79.2G    15%    /home
/dev/sd0d      3.9G    220K    3.7G     0%    /tmp
/dev/sd0f      5.8G    2.2G    3.4G    39%    /usr
/dev/sd0g      986M    248M    688M    26%    /usr/X11R6
/dev/sd0h     19.4G    3.8G   14.6G    21%    /usr/local
/dev/sd0j      5.8G    241M    5.3G     4%    /usr/obj
/dev/sd0i      2.9G    1.8G   1014M    64%    /usr/src
/dev/sd0e      5.8G   32.3M    5.5G     1%    /var
```


The numbers `disklabel(8)` shows are sector counts, and a sector
is 512 bytes (because diskabel said so, though in my experience,
drive firmware almost always reports 512).

```
dc 2k GB 2 1024 1024 * * * p       => blocks
dc 2k BLOCKS 2 / 1024 / 1024 / p   => GB
```

We can even alias this in tcsh

```
> alias GBtoS 'echo "5k \!:1 2 1024 1024 * * * p" | dc'
> alias StoGB 'echo "5k \!:1 2 / 1024 / 1024 / p" | dc'
> GBtoS 1
2097152
> StoGB 2097152
1.00000
```

I wouldn't stick that in my user's rc, but it's handy while we're figuring out these counts. The trick is to divide/multiply by 2 and ignore the last 6 digits.



let's try automatic mode

```
disklabel -E sd2
sd2> A
sd2*> p
OpenBSD area: 64-234441647; size: 234441583; free: 31
#                size           offset  fstype [fsize bsize   cpg]
  a:          2097152               64  4.2BSD   2048 16384     1 # /
  b:          2599568          2097216    swap                    
  c:        234441647                0  unused                    
  d:          8388576          4696800  4.2BSD   2048 16384     1 # /tmp
  e:         12539168         13085376  4.2BSD   2048 16384     1 # /var
  f:         12582912         25624544  4.2BSD   2048 16384     1 # /usr
  g:          2097152         38207456  4.2BSD   2048 16384     1 # /usr/X11R6
  h:         33889696         40304608  4.2BSD   2048 16384     1 # /usr/local
  i:          6291456         74194304  4.2BSD   2048 16384     1 # /usr/src
  j:         12582912         80485760  4.2BSD   2048 16384     1 # /usr/obj
  k:        141372960         93068672  4.2BSD   2048 16384     1 # /home
```

Close, but let's say I do not like that. I think I'll round down /var to 4GB and /usr/local to 16GB. That should give me almost 2 gigs for /home. I could also decide I want to consolidate `a`, `f` and `g` and sum up their sizes into a single big a partition.

delete everything with `z`


```
disklabel -E sd2
Label editor (enter '?' for help at any prompt)
sd2> z
sd2*> w
sd2> p
OpenBSD area: 64-234441647; size: 234441583; free: 234441583
#                size           offset  fstype [fsize bsize   cpg]
  c:        234441647                0  unused
sd2> a a
offset: [64] 
size: [234441583] 2097152
FS type: [4.2BSD] 
sd2*> a b
offset: [2097216] 
size: [232344431] 2599568
FS type: [swap] swap
sd2*> a d
offset: [4696784] 
size: [229744863] 8388576
FS type: [4.2BSD] 
sd2*> a e
offset: [13085344] 
size: [221356303] 8388608
FS type: [4.2BSD] 
sd2*> a f
offset: [21473952] 12582912
'f' aligned offset 12582912 lies outside the OpenBSD bounds or inside another partition
sd2*> a f
offset: [21473952] 
size: [212967695] 12582912
FS type: [4.2BSD] 
sd2*> a g
offset: [34056864] 
size: [200384783] 2097152
FS type: [4.2BSD] 
sd2*> a h
offset: [36154016] 
size: [198287631] 33554432
FS type: [4.2BSD] 
sd2*> a i
offset: [69708448] 
size: [164733199] 6291456
FS type: [4.2BSD] 
sd2*> a j
offset: [75999904] 
size: [158441743] 12582912
FS type: [4.2BSD] 
sd2*> a k
offset: [88582816] 
size: [145858831] 
FS type: [4.2BSD] 
sd2*> p
OpenBSD area: 64-234441647; size: 234441583; free: 31
#                size           offset  fstype [fsize bsize   cpg]
  a:          2097152               64  4.2BSD   2048 16384     1 
  b:          2599568          2097216    swap                    
  c:        234441647                0  unused                    
  d:          8388544          4696800  4.2BSD   2048 16384     1 
  e:          8388608         13085344  4.2BSD   2048 16384     1 
  f:         12582912         21473952  4.2BSD   2048 16384     1 
  g:          2097152         34056864  4.2BSD   2048 16384     1 
  h:         33554432         36154016  4.2BSD   2048 16384     1 
  i:          6291456         69708448  4.2BSD   2048 16384     1 
  j:         12582912         75999904  4.2BSD   2048 16384     1 
  k:        145858816         88582816  4.2BSD   2048 16384     1 
sd2*> w
```



```
doas disklabel sd2
# /dev/rsd2c:
type: SCSI
disk: SCSI disk
label: sage 3639S
duid: f511cf504ce61e02
flags:
bytes/sector: 512
sectors/track: 63
tracks/cylinder: 255
sectors/cylinder: 16065
cylinders: 14593
total sectors: 234441647
boundstart: 64
boundend: 234441647
drivedata: 0 

16 partitions:
#                size           offset  fstype [fsize bsize   cpg]
  a:          2097152               64  4.2BSD   2048 16384     1 
  b:          2599568          2097216    swap                    
  c:        234441647                0  unused                    
  d:          8388544          4696800  4.2BSD   2048 16384     1 
  e:          8388608         13085344  4.2BSD   2048 16384     1 
  f:         12582912         21473952  4.2BSD   2048 16384     1 
  g:          2097152         34056864  4.2BSD   2048 16384     1 
  h:         33554432         36154016  4.2BSD   2048 16384     1 
  i:          6291456         69708448  4.2BSD   2048 16384     1 
  j:         12582912         75999904  4.2BSD   2048 16384     1 
  k:        145858816         88582816  4.2BSD   2048 16384     1
```




now create all partitions...

```
newfs sd2[ad-k]
```

```
/# newfs sd2a
/dev/rsd2a: 1024.0MB in 2097152 sectors of 512 bytes
6 cylinder groups of 202.50MB, 12960 blocks, 25920 inodes each
super-block backups (for fsck -b #) at:
 160, 414880, 829600, 1244320, 1659040, 2073760,
/# newfs sd2d
/dev/rsd2d: 4096.0MB in 8388544 sectors of 512 bytes
21 cylinder groups of 202.50MB, 12960 blocks, 25920 inodes each
super-block backups (for fsck -b #) at:
 160, 414880, 829600, 1244320, 1659040, 2073760, 2488480, 2903200, 3317920,
 3732640, 4147360, 4562080, 4976800, 5391520, 5806240, 6220960, 6635680,
 7050400, 7465120, 7879840, 8294560,
/# newfs sd2e
/dev/rsd2e: 4096.0MB in 8388608 sectors of 512 bytes
21 cylinder groups of 202.50MB, 12960 blocks, 25920 inodes each
super-block backups (for fsck -b #) at:
 160, 414880, 829600, 1244320, 1659040, 2073760, 2488480, 2903200, 3317920,
 3732640, 4147360, 4562080, 4976800, 5391520, 5806240, 6220960, 6635680,
 7050400, 7465120, 7879840, 8294560,
/# newfs sd2f
/dev/rsd2f: 6144.0MB in 12582912 sectors of 512 bytes
31 cylinder groups of 202.50MB, 12960 blocks, 25920 inodes each
super-block backups (for fsck -b #) at:
 160, 414880, 829600, 1244320, 1659040, 2073760, 2488480, 2903200, 3317920,
 3732640, 4147360, 4562080, 4976800, 5391520, 5806240, 6220960, 6635680,
 7050400, 7465120, 7879840, 8294560, 8709280, 9124000, 9538720, 9953440,
 10368160, 10782880, 11197600, 11612320, 12027040, 12441760,
/# newfs sd2g
/dev/rsd2g: 1024.0MB in 2097152 sectors of 512 bytes
6 cylinder groups of 202.50MB, 12960 blocks, 25920 inodes each
super-block backups (for fsck -b #) at:
 160, 414880, 829600, 1244320, 1659040, 2073760,
/# newfs sd2h
/dev/rsd2h: 16384.0MB in 33554432 sectors of 512 bytes
81 cylinder groups of 202.50MB, 12960 blocks, 25920 inodes each
super-block backups (for fsck -b #) at:
 160, 414880, 829600, 1244320, 1659040, 2073760, 2488480, 2903200, 3317920,
 3732640, 4147360, 4562080, 4976800, 5391520, 5806240, 6220960, 6635680,
 7050400, 7465120, 7879840, 8294560, 8709280, 9124000, 9538720, 9953440,
 10368160, 10782880, 11197600, 11612320, 12027040, 12441760, 12856480,
 13271200, 13685920, 14100640, 14515360, 14930080, 15344800, 15759520,
 16174240, 16588960, 17003680, 17418400, 17833120, 18247840, 18662560,
 19077280, 19492000, 19906720, 20321440, 20736160, 21150880, 21565600,
 21980320, 22395040, 22809760, 23224480, 23639200, 24053920, 24468640,
 24883360, 25298080, 25712800, 26127520, 26542240, 26956960, 27371680,
 27786400, 28201120, 28615840, 29030560, 29445280, 29860000, 30274720,
 30689440, 31104160, 31518880, 31933600, 32348320, 32763040, 33177760,
/# newfs sd2i
/dev/rsd2i: 3072.0MB in 6291456 sectors of 512 bytes
16 cylinder groups of 202.50MB, 12960 blocks, 25920 inodes each
super-block backups (for fsck -b #) at:
 160, 414880, 829600, 1244320, 1659040, 2073760, 2488480, 2903200, 3317920,
 3732640, 4147360, 4562080, 4976800, 5391520, 5806240, 6220960,
/# newfs sd2j
/dev/rsd2j: 6144.0MB in 12582912 sectors of 512 bytes
31 cylinder groups of 202.50MB, 12960 blocks, 25920 inodes each
super-block backups (for fsck -b #) at:
 160, 414880, 829600, 1244320, 1659040, 2073760, 2488480, 2903200, 3317920,
 3732640, 4147360, 4562080, 4976800, 5391520, 5806240, 6220960, 6635680,
 7050400, 7465120, 7879840, 8294560, 8709280, 9124000, 9538720, 9953440,
 10368160, 10782880, 11197600, 11612320, 12027040, 12441760,
/# newfs sd2k
/dev/rsd2k: 71220.1MB in 145858816 sectors of 512 bytes
352 cylinder groups of 202.50MB, 12960 blocks, 25920 inodes each
super-block backups (for fsck -b #) at:
 160, 414880, 829600, 1244320, 1659040, 2073760, 2488480, 2903200, 3317920,
 3732640, 4147360, 4562080, 4976800, 5391520, 5806240, 6220960, 6635680,
 7050400, 7465120, 7879840, 8294560, 8709280, 9124000, 9538720, 9953440,
 10368160, 10782880, 11197600, 11612320, 12027040, 12441760, 12856480,
 13271200, 13685920, 14100640, 14515360, 14930080, 15344800, 15759520,
 16174240, 16588960, 17003680, 17418400, 17833120, 18247840, 18662560,
 19077280, 19492000, 19906720, 20321440, 20736160, 21150880, 21565600,
 21980320, 22395040, 22809760, 23224480, 23639200, 24053920, 24468640,
 24883360, 25298080, 25712800, 26127520, 26542240, 26956960, 27371680,
 27786400, 28201120, 28615840, 29030560, 29445280, 29860000, 30274720,
 30689440, 31104160, 31518880, 3193300, 32348320, 32763040, 33177760,
 33592480, 34007200, 34421920, 34836640, 35251360, 35666080, 36080800,
 36495520, 36910240, 37324960, 37739680, 38154400, 38569120, 38983840,
 39398560, 39813280, 40228000, 40642720, 41057440, 41472160, 41886880,
 42301600, 42716320, 43131040, 43545760, 43960480, 44375200, 44789920,
 45204640, 45619360, 46034080, 46448800, 46863520, 47278240, 47692960,
 48107680, 48522400, 48937120, 49351840, 49766560, 50181280, 50596000,
 51010720, 51425440, 51840160, 52254880, 52669600, 53084320, 53499040,
 53913760, 54328480, 54743200, 55157920, 55572640, 55987360, 56402080,
 56816800, 57231520, 57646240, 58060960, 58475680, 58890400, 59305120,
 59719840, 60134560, 60549280, 60964000, 61378720, 61793440, 62208160,
 62622880, 63037600, 63452320, 63867040, 64281760, 64696480, 65111200,
 65525920, 65940640, 66355360, 66770080, 67184800, 67599520, 68014240,
 68428960, 68843680, 69258400, 69673120, 70087840, 70502560, 70917280,
 71332000, 71746720, 72161440, 72576160, 72990880, 73405600, 73820320,
 74235040, 74649760, 75064480, 75479200, 75893920, 76308640, 76723360,
 77138080, 77552800, 77967520, 78382240, 78796960, 79211680, 79626400,
 80041120, 80455840, 80870560, 81285280, 81700000, 82114720, 82529440,
 82944160, 83358880, 83773600, 84188320, 84603040, 85017760, 85432480,
 85847200, 86261920, 86676640, 87091360, 87506080, 87920800, 88335520,
 88750240, 89164960, 89579680, 89994400, 90409120, 90823840, 91238560,
 91653280, 92068000, 92482720, 92897440, 93312160, 93726880, 94141600,
 94556320, 94971040, 95385760, 95800480, 96215200, 96629920, 97044640,
 97459360, 97874080, 98288800, 98703520, 99118240, 99532960, 99947680,
 100362400, 100777120, 101191840, 101606560, 102021280, 102436000, 102850720,
 103265440, 103680160, 104094880, 104509600, 104924320, 105339040, 105753760,
 106168480, 106583200, 106997920, 107412640, 107827360, 108242080, 108656800,
 109071520, 109486240, 109900960, 110315680, 110730400, 111145120, 111559840,
 111974560, 112389280, 112804000, 113218720, 113633440, 114048160, 114462880,
 114877600, 115292320, 115707040, 116121760, 116536480, 116951200, 117365920,
 117780640, 118195360, 118610080, 119024800, 119439520, 119854240, 120268960,
 120683680, 121098400, 121513120, 121927840, 122342560, 122757280, 123172000,
 123586720, 124001440, 124416160, 124830880, 125245600, 125660320, 126075040,
 126489760, 126904480, 127319200, 127733920, 128148640, 128563360, 128978080,
 129392800, 129807520, 130222240, 130636960, 131051680, 131466400, 131881120,
 132295840, 132710560, 133125280, 133540000, 133954720, 134369440, 134784160,
 135198880, 135613600, 136028320, 136443040, 136857760, 137272480, 137687200,
 138101920, 138516640, 138931360, 139346080, 139760800, 140175520, 140590240,
 141004960, 141419680, 141834400, 142249120, 142663840, 143078560, 143493280,
 143908000, 144322720, 144737440, 145152160, 145566880
```

Let's test


```
jakkal# mount /dev/sd2a /mnt
jakkal# cd /mnt
/mnt# ls
/mnt# ls -a
.  ..
/mnt# df -h
Filesystem     Size    Used   Avail Capacity  Mounted on
/dev/sd0a      986M    515M    422M    55%    /
/dev/sd0k     97.7G   13.6G   79.2G    15%    /home
/dev/sd0d      3.9G    220K    3.7G     0%    /tmp
/dev/sd0f      5.8G    2.2G    3.4G    39%    /usr
/dev/sd0g      986M    248M    688M    26%    /usr/X11R6
/dev/sd0h     19.4G    3.8G   14.6G    21%    /usr/local
/dev/sd0j      5.8G    241M    5.3G     4%    /usr/obj
/dev/sd0i      2.9G    1.8G   1014M    64%    /usr/src
/dev/sd0e      5.8G   32.3M    5.5G     1%    /var
/dev/sd2a      986M    2.0K    937M     0%    /mnt
/mnt# echo hi > hi
/mnt# cat hi
hi
```


cool

```
cd / ; umount /mnt
```

let's try to copy stuff

```
/# mount /dev/sd2e /mnt
/# df -h
Filesystem     Size    Used   Avail Capacity  Mounted on
/dev/sd0a      986M    515M    422M    55%    /
/dev/sd0k     97.7G   13.6G   79.2G    15%    /home
/dev/sd0d      3.9G    198K    3.7G     0%    /tmp
/dev/sd0f      5.8G    2.2G    3.4G    39%    /usr
/dev/sd0g      986M    248M    688M    26%    /usr/X11R6
/dev/sd0h     19.4G    3.8G   14.6G    21%    /usr/local
/dev/sd0j      5.8G    248M    5.3G     4%    /usr/obj
/dev/sd0i      2.9G    1.8G   1014M    64%    /usr/src
/dev/sd0e      5.8G   32.2M    5.5G     1%    /var
/dev/sd2e      3.9G    2.0K    3.7G     0%    /mnt

cd /mnt
/mnt# dump -0f - /var/ | restore -rf -
  DUMP: Dumping sub files/directories from /var
  DUMP: Dumping file/directory /var/
  DUMP: Date of this level 0 dump: Tue Oct 18 20:09:06 2022
  DUMP: Date of last level 0 dump: the epoch
  DUMP: Dumping /dev/rsd0e (/var) to standard output
  DUMP: mapping (Pass I) [regular files]
  DUMP: mapping (Pass II) [directories]
  DUMP: estimated 42447 tape blocks.
  DUMP: Volume 1 started at: Tue Oct 18 20:09:07 2022
  DUMP: dumping (Pass III) [directories]
  DUMP: dumping (Pass IV) [regular files]
  DUMP: 35015 tape blocks
  DUMP: Date of this level 0 dump: Tue Oct 18 20:09:06 2022
  DUMP: Volume 1 completed at: Tue Oct 18 20:09:18 2022
  DUMP: Volume 1 took 0:00:11
  DUMP: Volume 1 transfer rate: 3183 KB/s
  DUMP: Date this dump completed:  Tue Oct 18 20:09:18 2022
  DUMP: Average transfer rate: 3183 KB/s
  DUMP: DUMP IS DONE
```

that creates a restoresymtable which is used for incremental restores;
-0 effectively means full back
each higher level means "backup everything since the last backup of a lower level"
so you would restore backup level 0 and continue with 1, 2... up to how many levels your backup policy has. But that's besides the point, I'm only using it to copy paste the file systems

```
/mnt# ls -lh restoresymtable 
-rw-------  1 root  wheel   766K Oct 18 20:09 restoresymtable
```


anyway, let's re-wipe this; we need to go to singleuser mode to do this properly

```
# cd / ; umount /mnt
# newfs sd2e
/dev/rsd2e: 4096.0MB in 8388608 sectors of 512 bytes
21 cylinder groups of 202.50MB, 12960 blocks, 25920 inodes each
super-block backups (for fsck -b #) at:
 160, 414880, 829600, 1244320, 1659040, 2073760, 2488480, 2903200, 3317920,
 3732640, 4147360, 4562080, 4976800, 5391520, 5806240, 6220960, 6635680,
 7050400, 7465120, 7879840, 8294560,
```



now, we should reboot into single user mode with nothing mounted and nothing going on so that we can get a good snapshop. `reboot`, then `boot -s`

start with `/` to get it out of the way.

```
mount sd2a /mnt
cd /mnt
dump -0f - / | restore -rf - 
sed -i /mnt/etc/fstab -e 's/db0061b2fe09e6a0/f511cf504ce61e02/'
cd /
umount /mnt
```

We need to adjust the new `/etc/fstab` to actually mount the partitions on the new disk.

Dump creates `restoresymtable` which we don't need since we only ever restore the backup, but it helps to make sure when we boot into this later, it actually mounts the correct disk's partitions (e.g. we didn't forget to adjust the fstab). We'll delete those later.

Skip `b` (swap), skip `c` (full disk), skip `d` (`/tmp`), continue from `e`

repeat:

```
mount /dev/sd2e /mnt
cd /mnt
dump -0f - /var | restore -rf - 
cd /
umount
```

And so on.

Let's test the boot. `reboot`, then `boot sd2a:/bsd`

----

Cool. Let's swap the disks IRL...

Looking at the bottom of the laptop, it looks like there are a couple of Torx screws to get out of the way (because I have to justify having bought a couple of sets of Torx heads over the years), and the maintenance manual confirms this. The maintenance manual also says to *"carefully remove the Hard Drive cover as shown in the figure"* but it doesn't actually show how. If you have such a laptop and you don't want to break the cover (which includes shielding), you're supposed to remove if inward-to-outward (pull up from the VGA port side) while gently pulling it towards the longer edge (that frees up the last tab on the shorter edge), and then you can pull it out towards the VGA port side.

The old disk just slides out (fun fact -- the maintenance manual mentioned every other paragraph to not press down on the "middle of the hard drive" to not damage the platter or head). It's in some sort of caddy, imprisoned by 4 philips head screws. Those come right out, SSD goes in, SSD slides in, cover is reinstalled, we should be good to go.

TODO integrate pictures and whatnot

----

Aaaaaand we run into a bit of a wall. My Bios doesn't "see" the SSD. Much googling has given me some pointers, like flip between "IDE" and "AHCI" SATA modes in the BIOS, unfortunately, I cannot seem to enter BIOS settings (it just reboots when the BIOS setup menu is supposed to show up). This laptop used to have a Windows 7 utility called Toshiba HWSetup, which is long gone. I'll try to get windows on some USB drive to try to get some version of that installed and maybe it lets me switch between those modes, or at least disable PXEBOOT or maybe something else (foreshadowing...)

So, I know the drive boots off of the SATA-to-USB interface, I know it boots off of the SATA-to-USB interface without the old hard drive in there, and I know I can boot off of a usb thumb drive (which is how I installed OpenBSD on this thing in the first place).

Booting off of a USB pen drive with the installation media on it (`install71.img`), it does not give me the option to boot off of the SSD either (it only lists hd0 which is the USB stick itself). But, if I continue to boot it and press U for `(U)pdate`, it does prompt me about `sd0`. Escaping to a shell and fiddling about, I can tell the USB stick itself is now `sd2`. So everything seems fine once the kernel is booted.

Looking through `boot(8)`, I see we can ask the bootloader to _ask_ where the root partition is. Let's try that! Reboot, at the prompt type `boot -a`. When prompted, tell it `sd0a` for root and `sd0b` for swap. It starts to boot, but hangs after starting `ntpd`. Okay, maybe `/boot.rd` isn't the best image to actually load the operating system (which is the default image for the installation media). 

Let's try with `/bsd`. Reboot, say `boot /bsd -a`, when prompted for the root partition, tell it `sd0a` for root and `sd0b` for swap and... it almost boots correctly. There are no additional vttys (or whatever BSD calls them) installed, so X won't start. Network seems to work. There were a whole lot more devices showing up as "not configured" in `dmesg`, but maybe that's because the kernel on the installation media doesn't have a whole much of anything in it. Maybe the installer configures the kernel it's installing resulting in a kernel that can actually configure all of the devices. 

Since we're now sort-of booted into my usual system, let's hack!

```sh
mount /dev/sd2a /mnt
mv /mnt/bsd /mnt/bsd.installer
cp /bsd /mnt/bsd
```

Reading about `boot.conf` in the man pages, I see I can specify some things to have a few characters less to type:

```sh
cat <<EOF > /mnt/etc/boot.conf
set image /bsd
set howto -a
EOF
```

Now let's get out of here:

```sh
cd /
umount /mnt
reboot
```

I've searched the whole wide world (wide web) if I can tell the bootloader to have `root on sd0a swap on sd0b dump on sd0b`, but it turns out you can only do that when recompiling the kernel. I'll wait on that since recompiling the kernel on this ancient netbook takes at least two hours. But, now that it's back at the `boot>` prompt, let's hit enter and see what happens.

It reads the new `/etc/boot.conf` and uses the new `/bsd` kernel which we copied over from our SSD, and then asks for the root device: `sd0a`, enter, aaaaand... Yes, we're booted off of the SSD, X is running, I can log in, everything is mounted, and we can remove the USB stick.

So, we are in 2022 and I have a system that effectively needs a boot "floppy". Now, I just need to gather the necessary willpower to recompile the kernel (again) with `conf bsd root on sd0a swap on sd0b dump on sd0b` and I don't have to type `sd0a<CR><CR>` every time I boot now.

The only additional annoyance is that if I hibernate the system, the BIOS knows it was hibernated and disables the boot menu and for some strange reason `lan` (PXE) takes precedence over USB. This means I need to `CTRL+ALT+DEL` out of the failing PXE boot 3 times until it will let me tell it to use the USB. Then, it proceeds to boot asking for root, and then proceeds to detect the hibernation image and load it. And everything is fine afterwards.

Why would I torture myself this way with a boot floppy-actually-usb-drive? I have a feeling that old hard drive will give up the ghost one of these days, while the SSD definitely has a few years' worth of life in it. And I'll have to check the benchmarks, maybe we do get some speed improvements.

----

Finally, delete those `restoresymtable` files: (I'm feeling courages and using shell expansions as root, woooo!)

```
doas rm -f {/,/var/,/home/,/usr/,/usr/X11R6/,/usr/local/,/usr/obj/,/usr/src/}restoresymtable
```



Benchmarks?
-----------

I ran this:

```
doas fio --name=read --iodepth=1 --rw=read --bs=4k --direct=0 --size=512M --numjobs=2 --runtime=240 --group_reporting
doas fio --name=write --iodepth=1 --rw=write --bs=4k --direct=0 --size=512M --numjobs=2 --runtime=240 --group_reporting
doas fio --name=randread --iodepth=1 --rw=randread --bs=4k --direct=0 --size=128MB --numjobs=2 --runtime=240 --group_reporting
doas fio --name=randrw --iodepth=1 --rw=randrw --bs=4k --direct=0 --size=128MB --numjobs=2 --runtime=240 --group_reporting
```

I'm not good at disk benchmarks, but usually they test sequential and random reads and writes. 4k should be a block (I don't know how to check), `numjobs` is 2 because I have a single hyperthreaded core, `iodepth=1` because I'm not sure if higher depths are supported without `libaio` or if it matters on SATA. Shrugs, at least I ran the same tests on both disks.

Oh, if you run those, it _will_ create files named something like `randrw.0.0` which are 128/512MB. Don't forget to delete them.

Here's a table:

| Disk            | Sequential read | sequential write  | random read   | random rw                     |
|:---------------:|-----------------|-------------------|---------------|-------------------------------|
| **spinny disk** | 10.3 MB/s       | 30.3 MB/s         | 483 KB/s      | 257-479 KB/s read, 280-482 KB/s write |
| **ssd**		  |					|					|				|                               |

### Before

Here's the before (with the old spinny disk installed):

#### Sequential read

```
read: (g=0): rw=read, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=psync, iodepth=1
...
fio-3.26
Starting 2 threads
Jobs: 2 (f=2): [R(2)][100.0%][r=25.2MiB/s][r=6446 IOPS][eta 00m:00s]
read: (groupid=0, jobs=2): err= 0: pid=2138416428: Wed Oct 19 21:19:38 2022
  read: IOPS=2646, BW=10.3MiB/s (10.8MB/s)(1024MiB/99064msec)
    clat (usec): min=23, max=88329, avg=718.63, stdev=4160.79
     lat (usec): min=29, max=88335, avg=726.04, stdev=4160.66
    clat percentiles (usec):
     |  1.00th=[   25],  5.00th=[   26], 10.00th=[   27], 20.00th=[   27],
     | 30.00th=[   32], 40.00th=[   33], 50.00th=[   34], 60.00th=[   35],
     | 70.00th=[   39], 80.00th=[   41], 90.00th=[   85], 95.00th=[  388],
     | 99.00th=[21627], 99.50th=[32637], 99.90th=[43779], 99.95th=[43779],
     | 99.99th=[54789]
   bw (  KiB/s): min= 3238, max=109169, per=99.44%, avg=10526.60, stdev=5844.51, samples=394
   iops        : min=  808, max=27291, avg=2630.99, stdev=1461.18, samples=394
  lat (usec)   : 50=88.09%, 100=2.75%, 250=0.40%, 500=4.18%, 750=0.08%
  lat (usec)   : 1000=1.21%
  lat (msec)   : 2=0.25%, 4=0.15%, 10=0.17%, 20=0.95%, 50=1.76%
  lat (msec)   : 100=0.01%
  cpu          : usr=3.26%, sys=8.55%, ctx=15871, majf=0, minf=6
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=262144,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
   READ: bw=10.3MiB/s (10.8MB/s), 10.3MiB/s-10.3MiB/s (10.8MB/s-10.8MB/s), io=1024MiB (1074MB), run=99064-99064msec
```

#### Sequential write:

```
write: (g=0): rw=write, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=psync, iodepth=1
...
fio-3.26
Starting 2 threads
write: Laying out IO file (1 file / 512MiB)
write: Laying out IO file (1 file / 512MiB)
Jobs: 2 (f=2): [W(2)][97.1%][w=26.3MiB/s][w=6735 IOPS][eta 00m:01s]
write: (groupid=0, jobs=2): err= 0: pid=1676831276: Wed Oct 19 21:28:13 2022
  write: IOPS=7754, BW=30.3MiB/s (31.8MB/s)(1024MiB/33807msec); 0 zone resets
    clat (usec): min=28, max=152399, avg=213.93, stdev=2318.41
     lat (usec): min=35, max=152407, avg=223.02, stdev=2318.37
    clat percentiles (usec):
     |  1.00th=[   36],  5.00th=[   43], 10.00th=[   47], 20.00th=[   48],
     | 30.00th=[   51], 40.00th=[   57], 50.00th=[   73], 60.00th=[   97],
     | 70.00th=[  119], 80.00th=[  128], 90.00th=[  159], 95.00th=[  180],
     | 99.00th=[  277], 99.50th=[  537], 99.90th=[35390], 99.95th=[43779],
     | 99.99th=[98042]
   bw (  KiB/s): min=19416, max=44486, per=100.00%, avg=31041.66, stdev=2323.90, samples=134
   iops        : min= 4853, max=11119, avg=7759.90, stdev=580.96, samples=134
  lat (usec)   : 50=29.27%, 100=33.37%, 250=36.07%, 500=0.79%, 750=0.03%
  lat (usec)   : 1000=0.01%
  lat (msec)   : 2=0.01%, 4=0.01%, 10=0.05%, 20=0.16%, 50=0.22%
  lat (msec)   : 100=0.03%, 250=0.01%
  cpu          : usr=11.37%, sys=37.57%, ctx=2689, majf=0, minf=1
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,262144,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
  WRITE: bw=30.3MiB/s (31.8MB/s), 30.3MiB/s-30.3MiB/s (31.8MB/s-31.8MB/s), io=1024MiB (1074MB), run=33807-33807msec
```

#### Random read:

```
randread: (g=0): rw=randread, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=psync, iodepth=1
...
fio-3.26
Starting 2 threads
Jobs: 2 (f=2): [r(2)][100.0%][r=552KiB/s][r=138 IOPS][eta 00m:00s]
randread: (groupid=0, jobs=2): err= 0: pid=1071443244: Wed Oct 19 21:32:56 2022
  read: IOPS=109, BW=438KiB/s (448kB/s)(103MiB/240024msec)
    clat (usec): min=23, max=88032, avg=18230.89, stdev=14964.63
     lat (usec): min=29, max=88039, avg=18237.47, stdev=14964.61
    clat percentiles (usec):
     |  1.00th=[   27],  5.00th=[   30], 10.00th=[   35], 20.00th=[   37],
     | 30.00th=[   39], 40.00th=[17433], 50.00th=[23200], 60.00th=[26346],
     | 70.00th=[29230], 80.00th=[32113], 90.00th=[35914], 95.00th=[39060],
     | 99.00th=[43254], 99.50th=[44827], 99.90th=[46924], 99.95th=[47973],
     | 99.99th=[62653]
   bw (  KiB/s): min=  214, max=  856, per=99.80%, avg=437.82, stdev=56.06, samples=958
   iops        : min=   52, max=  214, avg=109.43, stdev=14.03, samples=958
  lat (usec)   : 50=36.07%, 100=0.57%, 250=0.16%, 500=0.02%
  lat (msec)   : 4=0.02%, 10=0.38%, 20=5.84%, 50=56.91%, 100=0.03%
  cpu          : usr=0.18%, sys=0.49%, ctx=16656, majf=0, minf=5
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=26276,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
   READ: bw=438KiB/s (448kB/s), 438KiB/s-438KiB/s (448kB/s-448kB/s), io=103MiB (108MB), run=240024-240024msec
```

#### Random readwrite:

```
randrw: (g=0): rw=randrw, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=psync, iodepth=1
...
fio-3.26
Starting 2 threads
randrw: Laying out IO file (1 file / 128MiB)
randrw: Laying out IO file (1 file / 128MiB)
Jobs: 2 (f=2): [m(2)][100.0%][r=648KiB/s,w=712KiB/s][r=162,w=178 IOPS][eta 00m:00s]
randrw: (groupid=0, jobs=2): err= 0: pid=1888610348: Wed Oct 19 21:41:06 2022
  read: IOPS=64, BW=257KiB/s (263kB/s)(60.2MiB/240020msec)
    clat (usec): min=23, max=2606.6k, avg=15257.28, stdev=48556.36
     lat (usec): min=29, max=2606.6k, avg=15263.83, stdev=48556.46
    clat percentiles (usec):
     |  1.00th=[     27],  5.00th=[     29], 10.00th=[     35],
     | 20.00th=[     36], 30.00th=[     37], 40.00th=[     38],
     | 50.00th=[     41], 60.00th=[  15401], 70.00th=[  20841],
     | 80.00th=[  24511], 90.00th=[  28443], 95.00th=[  32113],
     | 99.00th=[ 263193], 99.50th=[ 375391], 99.90th=[ 505414],
     | 99.95th=[ 557843], 99.99th=[1367344]
   bw (  KiB/s): min=   16, max=  769, per=100.00%, avg=273.99, stdev=87.06, samples=888
   iops        : min=    4, max=  192, avg=68.48, stdev=21.76, samples=888
  write: IOPS=64, BW=260KiB/s (266kB/s)(60.9MiB/240020msec); 0 zone resets
    clat (usec): min=31, max=4929.2k, avg=15623.81, stdev=64345.23
     lat (usec): min=38, max=4929.2k, avg=15630.99, stdev=64345.38
    clat percentiles (usec):
     |  1.00th=[     39],  5.00th=[     44], 10.00th=[     47],
     | 20.00th=[     50], 30.00th=[     53], 40.00th=[     64],
     | 50.00th=[     73], 60.00th=[  15664], 70.00th=[  21103],
     | 80.00th=[  24511], 90.00th=[  28705], 95.00th=[  32113],
     | 99.00th=[ 242222], 99.50th=[ 375391], 99.90th=[ 541066],
     | 99.95th=[ 633340], 99.99th=[3472884]
   bw (  KiB/s): min=   16, max=  769, per=100.00%, avg=280.23, stdev=85.96, samples=880
   iops        : min=    4, max=  192, avg=70.04, stdev=21.48, samples=880
  lat (usec)   : 50=37.28%, 100=18.65%, 250=0.45%, 500=0.02%, 750=0.02%
  lat (usec)   : 1000=0.01%
  lat (msec)   : 4=0.01%, 10=0.52%, 20=10.71%, 50=29.81%, 100=1.18%
  lat (msec)   : 250=0.36%, 500=0.86%, 750=0.12%, 1000=0.01%, 2000=0.01%
  lat (msec)   : >=2000=0.01%
  cpu          : usr=0.19%, sys=0.58%, ctx=13596, majf=0, minf=3
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=15414,15595,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
   READ: bw=257KiB/s (263kB/s), 257KiB/s-257KiB/s (263kB/s-263kB/s), io=60.2MiB (63.1MB), run=240020-240020msec
  WRITE: bw=260KiB/s (266kB/s), 260KiB/s-260KiB/s (266kB/s-266kB/s), io=60.9MiB (63.9MB), run=240020-240020msec
```

I figured I may as well try `iodepth=64` and it didn't seem to stick. But I did get better results the second time.

```
doas fio --name=randrw --iodepth=64 --rw=randrw --bs=4k --direct=0 --size=128M --numjobs=2 --runtime=240 --group_reporting
randrw: (g=0): rw=randrw, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=psync, iodepth=64
...
fio-3.26
Starting 2 threads
Jobs: 1 (f=1): [m(1),_(1)][72.2%][r=728KiB/s,w=800KiB/s][r=182,w=200 IOPS][eta 01m:33s]  
randrw: (groupid=0, jobs=2): err= 0: pid=1735393836: Wed Oct 19 22:00:06 2022
  read: IOPS=117, BW=468KiB/s (479kB/s)(110MiB/240074msec)
    clat (usec): min=22, max=7388.8k, avg=5958.35, stdev=54733.25
     lat (usec): min=28, max=7388.8k, avg=5964.81, stdev=54733.33
    clat percentiles (usec):
     |  1.00th=[    25],  5.00th=[    26], 10.00th=[    28], 20.00th=[    34],
     | 30.00th=[    35], 40.00th=[    35], 50.00th=[    36], 60.00th=[    37],
     | 70.00th=[    38], 80.00th=[    44], 90.00th=[ 13960], 95.00th=[ 23725],
     | 99.00th=[ 89654], 99.50th=[274727], 99.90th=[463471], 99.95th=[549454],
     | 99.99th=[834667]
   bw (  KiB/s): min=   16, max= 4889, per=100.00%, avg=859.73, stdev=406.90, samples=620
   iops        : min=    4, max= 1221, avg=214.77, stdev=101.65, samples=620
  write: IOPS=117, BW=471KiB/s (482kB/s)(110MiB/240074msec); 0 zone resets
    clat (usec): min=27, max=8351.6k, avg=6755.65, stdev=77544.85
     lat (usec): min=34, max=8351.6k, avg=6762.72, stdev=77544.93
    clat percentiles (usec):
     |  1.00th=[     35],  5.00th=[     39], 10.00th=[     42],
     | 20.00th=[     45], 30.00th=[     47], 40.00th=[     48],
     | 50.00th=[     50], 60.00th=[     57], 70.00th=[     65],
     | 80.00th=[     74], 90.00th=[  14353], 95.00th=[  23987],
     | 99.00th=[  93848], 99.50th=[ 308282], 99.90th=[ 492831],
     | 99.95th=[ 599786], 99.99th=[4664067]
   bw (  KiB/s): min=   16, max= 4969, per=100.00%, avg=860.65, stdev=400.65, samples=633
   iops        : min=    4, max= 1241, avg=214.99, stdev=100.09, samples=633
  lat (usec)   : 50=66.68%, 100=16.92%, 250=0.41%, 500=0.14%, 750=0.06%
  lat (usec)   : 1000=0.01%
  lat (msec)   : 2=0.01%, 4=0.12%, 10=2.72%, 20=6.20%, 50=5.20%
  lat (msec)   : 100=0.64%, 250=0.30%, 500=0.51%, 750=0.07%, 1000=0.01%
  lat (msec)   : 2000=0.01%, >=2000=0.01%
  cpu          : usr=0.39%, sys=1.05%, ctx=9246, majf=0, minf=7
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=28091,28255,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=64

Run status group 0 (all jobs):
   READ: bw=468KiB/s (479kB/s), 468KiB/s-468KiB/s (479kB/s-479kB/s), io=110MiB (115MB), run=240074-240074msec
  WRITE: bw=471KiB/s (482kB/s), 471KiB/s-471KiB/s (482kB/s-482kB/s), io=110MiB (116MB), run=240074-240074msec
```

### After

Here's the after (with the SSD installed):
