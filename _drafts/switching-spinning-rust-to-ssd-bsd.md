~> doas fdisk sd0
doas (jakkal@jakkal-mini.my.domain) password: 
Disk: sd0       geometry: 19457/255/63 [312581808 Sectors]
Offset: 0       Signature: 0xAA55
            Starting         Ending         LBA Info:
 #: id      C   H   S -      C   H   S [       start:        size ]
-------------------------------------------------------------------------------
 0: 00      0   0   0 -      0   0   0 [           0:           0 ] unused      
 1: 00      0   0   0 -      0   0   0 [           0:           0 ] unused      
 2: 00      0   0   0 -      0   0   0 [           0:           0 ] unused      
*3: A6      0   1   2 -  19457  80  63 [          64:   312581744 ] OpenBSD


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



doas fdisk -i sd2
y





fdisk
*3: A6      0   1   2 -  14593  80  62 [          64:   234441583 ] OpenBSD

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


dc 2k GB 2 1024 1024 * * * p       => blocks
dc 2k BLOCKS 2 / 1024 / 1024 / p   => GB
make /var sd2e smaller, 2GB, or 4194304




let's try automatic mode
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

DO NOT LIKE

delete everything with `z`


sd2*> z
sd2> a a
offset: [64] 
size: [234441583] 2097152
FS type: [4.2BSD] 
sd2*> a b
offset: [2097216] 
size: [232344431] 2599656
FS type: [swap] swap
sd2*> a d
offset: [4696872] 
size: [229744775] 8388576
FS type: [4.2BSD] 
sd2*> a e
offset: [13085440] 
size: [221356207] 8388608
FS type: [4.2BSD] 
sd2*> a f
offset: [21474048] 
size: [212967599] 12582912
FS type: [4.2BSD] 
sd2*> a g
offset: [34056960] 
size: [200384687] 2097152
FS type: [4.2BSD] 
sd2*> a h
offset: [36154112] 
size: [198287535] 33554432
FS type: [4.2BSD] 
sd2*> a j
offset: [69708544] 
size: [164733103] 12582912
FS type: [4.2BSD] 
sd2*> a k
offset: [82291456] 
size: [152150191] 
FS type: [4.2BSD]
sd2*> 2
sd2> p
OpenBSD area: 64-234441647; size: 234441583; free: 39
#                size           offset  fstype [fsize bsize   cpg]
  a:          2097152               64  4.2BSD   2048 16384     1 
  b:          2599656          2097216    swap                    
  c:        234441647                0  unused                    
  d:          8388544          4696896  4.2BSD   2048 16384     1 
  e:          8388608         13085440  4.2BSD   2048 16384     1 
  f:         12582912         21474048  4.2BSD   2048 16384     1 
  g:          2097152         34056960  4.2BSD   2048 16384     1 
  h:         33554432         36154112  4.2BSD   2048 16384     1 
  j:         12582912         69708544  4.2BSD   2048 16384     1 
  k:        152150176         82291456  4.2BSD   2048 16384     1 




~> doas disklabel sd2
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
  b:          2599656          2097216    swap                    
  c:        234441647                0  unused                    
  d:          8388544          4696896  4.2BSD   2048 16384     1 
  e:          8388608         13085440  4.2BSD   2048 16384     1 
  f:         12582912         21474048  4.2BSD   2048 16384     1 
  g:          2097152         34056960  4.2BSD   2048 16384     1 
  h:         33554432         36154112  4.2BSD   2048 16384     1 
  j:         12582912         69708544  4.2BSD   2048 16384     1 
  k:        152150176         82291456  4.2BSD   2048 16384     1 





now create all partitions...


jakkal# newfs sd2a
/dev/rsd2a: 1024.0MB in 2097152 sectors of 512 bytes
6 cylinder groups of 202.50MB, 12960 blocks, 25920 inodes each
super-block backups (for fsck -b #) at:
 160, 414880, 829600, 1244320, 1659040, 2073760,
jakkal# newfs sd2d
/dev/rsd2d: 4096.0MB in 8388544 sectors of 512 bytes
21 cylinder groups of 202.50MB, 12960 blocks, 25920 inodes each
super-block backups (for fsck -b #) at:
 160, 414880, 829600, 1244320, 1659040, 2073760, 2488480, 2903200, 3317920,
 3732640, 4147360, 4562080, 4976800, 5391520, 5806240, 6220960, 6635680,
 7050400, 7465120, 7879840, 8294560,
jakkal# newfs sd2e
/dev/rsd2e: 4096.0MB in 8388608 sectors of 512 bytes
21 cylinder groups of 202.50MB, 12960 blocks, 25920 inodes each
super-block backups (for fsck -b #) at:
 160, 414880, 829600, 1244320, 1659040, 2073760, 2488480, 2903200, 3317920,
 3732640, 4147360, 4562080, 4976800, 5391520, 5806240, 6220960, 6635680,
 7050400, 7465120, 7879840, 8294560,
jakkal# newfs sd2f
/dev/rsd2f: 6144.0MB in 12582912 sectors of 512 bytes
31 cylinder groups of 202.50MB, 12960 blocks, 25920 inodes each
super-block backups (for fsck -b #) at:
 160, 414880, 829600, 1244320, 1659040, 2073760, 2488480, 2903200, 3317920,
 3732640, 4147360, 4562080, 4976800, 5391520, 5806240, 6220960, 6635680,
 7050400, 7465120, 7879840, 8294560, 8709280, 9124000, 9538720, 9953440,
 10368160, 10782880, 11197600, 11612320, 12027040, 12441760,
jakkal# newfs sd2g
/dev/rsd2g: 1024.0MB in 2097152 sectors of 512 bytes
6 cylinder groups of 202.50MB, 12960 blocks, 25920 inodes each
super-block backups (for fsck -b #) at:
 160, 414880, 829600, 1244320, 1659040, 2073760,
jakkal# newfs sd2h
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
jakkal# newfs sd2i
newfs: sd2i: Device not configured
jakkal# newfs sd2j
/dev/rsd2j: 6144.0MB in 12582912 sectors of 512 bytes
31 cylinder groups of 202.50MB, 12960 blocks, 25920 inodes each
super-block backups (for fsck -b #) at:
 160, 414880, 829600, 1244320, 1659040, 2073760, 2488480, 2903200, 3317920,
 3732640, 4147360, 4562080, 4976800, 5391520, 5806240, 6220960, 6635680,
 7050400, 7465120, 7879840, 8294560, 8709280, 9124000, 9538720, 9953440,
 10368160, 10782880, 11197600, 11612320, 12027040, 12441760,
jakkal# newfs sd2k
/dev/rsd2k: 74292.1MB in 152150176 sectors of 512 bytes
367 cylinder groups of 202.50MB, 12960 blocks, 25920 inodes each
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
 143908000, 144322720, 144737440, 145152160, 145566880, 145981600, 146396320,
 146811040, 147225760, 147640480, 148055200, 148469920, 148884640, 149299360,
 149714080, 150128800, 150543520, 150958240, 151372960, 151787680,



notice there is no sd2i...
I know this is a "pseudo" partition for FAT stuff (or foreign partition types), but I couldn't find concrete explanations. Maybe I'll dive into OpenBSD's code and maybe I'll find a comment mentioning the WHYs. Moving on...

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


cool
cd / ; umount /mnt

let's try to copy stuff
mount /dev/sd2e /mnt
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

that creates a restoresymtable which is used for incremental restores;
-0 effectively means full back
each higher level means "backup everything since the last backup of a lower level"
so you would restore backup level 0 and continue with 1, 2... up to how many levels your backup policy has. But that's besides the point, I'm only using it to copy paste the file systems
/mnt# ls -lh restoresymtable 
-rw-------  1 root  wheel   766K Oct 18 20:09 restoresymtable


anyway, let's re-wipe this; we need to go to singleuser mode to do this properly

cd / ; umount /mnt
newfs sd2e
/dev/rsd2e: 4096.0MB in 8388608 sectors of 512 bytes
21 cylinder groups of 202.50MB, 12960 blocks, 25920 inodes each
super-block backups (for fsck -b #) at:
 160, 414880, 829600, 1244320, 1659040, 2073760, 2488480, 2903200, 3317920,
 3732640, 4147360, 4562080, 4976800, 5391520, 5806240, 6220960, 6635680,
 7050400, 7465120, 7879840, 8294560,






shhhhhhhhhooooot I forgot the /usr/src parittion (sd2i); TODO start over

and do these changes
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


/var e 4194304                      2GiB
/usr/local 33554432                 16GiB
/home ~147664416                    ~70GiB

this is a note for me, not for the blog
set /etc/mk.conf
WRKOBJDIR=/usr/obj/ports
chown jakkal /usr/obj/ports
