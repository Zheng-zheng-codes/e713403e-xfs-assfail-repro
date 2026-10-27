# xfs: "Log failed size checks" triggers assertion in xfs_log_mount()

Reproduction materials for a syzkaller report:

```
kernel BUG in assfail (fs/xfs/xfs_message.c:102)
```

## Status

- Trigger: executing the stored syzkaller testcases
  (`repro/repro.kernsight.prog` then `repro/repro.prog`) on
  Linux 7.3.0-kernsight-kasan triggers the assertion in `xfs_log_mount()`.
- Reproduction count: 1/1 on a fresh VM.
- Taint: Not tainted.
- The failing check is the log size check
  (`XFS (loop0): AAIEEE! Log failed size checks. Abort!`), followed by
  `XFS: Assertion failed: 0, file: fs/xfs/xfs_log.c, line: 597`.
- XFS config: CONFIG_XFS_DEBUG=y, CONFIG_XFS_DEBUG_EXPENSIVE=y,
  CONFIG_XFS_ASSERT_FATAL=y.
- This is an observable assertion with a malformed image; it does not claim
  a new vulnerability or a specific root cause.

## Contents

```
repro/repro.kernsight.prog   state-construction recipe (~2.6 MB)
repro/repro.prog             crashing testcase (~24 KB; image embedded)
7.3/serial-7.3-20261024.log  full boot-to-crash serial log
7.3/crash-excerpt-7.3-20261024.txt
7.3/console-7.3-20261024.log
7.3/cmdline-7.3-20261024.txt
7.3/boot-info-7.3-20261024.txt
7.3/RESULT.txt
kernel/kernel-7.3.config, kernel/SOURCES.md
reproduce.sh, verify.sh, MANIFEST.sha256
```

## Reproduction

```
./reproduce.sh
```

Manual form (same VM; run the recipe first, then the testcase):

```
scp syz-execprog syz-executor repro/repro.kernsight.prog repro/repro.prog root@guest:/tmp/
ssh root@guest
cd /tmp && chmod +x syz-execprog syz-executor
./syz-execprog -procs=1 -repeat=1 -disable=swap -executor ./syz-executor ./repro.kernsight.prog
./syz-execprog -procs=1 -repeat=1 -disable=swap -executor ./syz-executor ./repro.prog
```

## Observed on 7.3.0

```
XFS (loop0): AAIEEE! Log failed size checks. Abort!
XFS: Assertion failed: 0, file: fs/xfs/xfs_log.c, line: 597
kernel BUG at fs/xfs/xfs_message.c:102!
RIP: 0010:assfail+0xa2/0xb0
 xfs_log_mount+0x4e3/0x590
 xfs_mountfs+0x10b9/0x2270
 xfs_fs_fill_super+0x15cc/0x20b0
Kernel panic - not syncing: Fatal exception
```
