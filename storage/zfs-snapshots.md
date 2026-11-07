# ZFS Snapshots

A snapshot is a point-in-time, copy-on-write copy of a ZFS dataset. It is instant and nearly space-free at creation, and it is the foundation of every TrueNAS backup strategy.

## When to Use
- You need a guaranteed-consistent copy before a risky operation (upgrade, migration, delete).
- You want cheap, frequent backups that cost almost no space.
- You are protecting against accidental deletion or corruption.

## How
```bash
zfs snapshot tank/data@2026-07-15   # take a snapshot
zfs list -t snapshot                 # list snapshots
zfs rollback tank/data@2026-07-15    # revert to it
```
The rollback destroys all changes made after the snapshot, so confirm before running it.

For scheduled backups, set a retention policy in the TrueNAS UI: Dataset → Snapshots.
- Daily: keep 5
- Weekly: keep 4
- Monthly: keep 2

## Rule
Snapshots are not backups — if the pool dies, snapshots die with it. Replicate for off-site protection.
