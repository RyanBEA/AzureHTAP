## Azure HTAP Async Scheduler Notes

- Provisioned VM: L8aos_v4 with Windows Server 2022 Gen2.
- Added Ultra Disk (512 GiB, 12 K IOPS, 2 GB/s throughput), mounted at `C:\HTAP\data` using NTFS mount point.
- Redirected HTAP temp/work directories via junction (`mklink /J C:\HTAP\HTAP-work C:\HTAP\data\work`), set `%TEMP%`/`%TMP%` to `C:\HTAP\data\temp`.
- Verified Ultra disk activity in Resource Monitor (`\Device\HarddiskVolume6`).
- Branch `feature/async-scheduler` updates `htap-prm.rb` with sliding queue dispatcher:
  - `launch_run_for_thread`: per-slot process setup/spawn.
  - `run_these_cases_async`: maintains pending queue, immediately backfills freed slots.
  - `process_completed_runs`: per-run finalization (JSON/CSV/LEEP export).
  - Added guard to legacy status comparisons (`section == "status"`).
- Commands executed:
  - `ruby htap-prm.rb -r <runfile> -t 7` (async run)
  - `git commit -m "Add async scheduler to keep HTAP runs continuous"`
  - `git push origin feature/async-scheduler`
- Observed status: `--> Batch #64 [Runs Active: 7/7, Runs done: 57 (0.02%), Errors: 0, ETA: 2.5 days] - Spawning threads`
- Next steps: validate outputs against baseline, refine ETA messaging, consider removing unused legacy batch code once confidence is high.
