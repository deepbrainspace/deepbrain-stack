Yes, you can absolutely split your setup that way—moving your restic backups from Cloudflare R2 to the regular IDrive plan ($70/5TB/year) and using IDrive e2 ($24.75/TB first year, $49.50/TB/year after) for S3-compatible backends like AppFlowy and Immich. It’s a smart way to leverage the strengths of both services while keeping costs low and maintaining S3 compatibility where needed. Let’s break it down to confirm it works for your needs and highlight how to make it happen.

### Splitting the Strategy
- **Regular IDrive ($70/5TB/year)**: This is the core IDrive backup service (Personal plan), designed for file-level backups from devices or servers. It’s not S3-compatible but integrates well with tools like restic for incremental, encrypted backups. At $70/year for 5 TB (or $35 first year with discounts), it’s dirt cheap—$14/TB/year normally, $7/TB first year.
- **IDrive e2 ($24.75/TB first year)**: S3-compatible object storage, perfect for AppFlowy and Immich, which need an S3 backend. Even at $49.50/TB/year post-first-year, it’s still a bargain compared to R2’s $180/TB/year.

Your plan:
- Move restic backups (currently on R2) to regular IDrive.
- Use e2 for AppFlowy and Immich S3 needs.

### Can It Work?
#### Restic to Regular IDrive
- **Compatibility**: Restic supports multiple backends, including R2 (S3) and SFTP/FTP, which regular IDrive offers via its “Server Backup” feature. IDrive provides SFTP access (e.g., `backup.idrive.com`, port 22) with your account credentials, so you can point restic to it instead of R2.
- **Setup**:
  1. Install restic if not already on your server (`sudo apt install restic`).
  2. Initialize a new repo on IDrive SFTP:
     ```bash
     restic -r sftp:user@backup.idrive.com:/path/to/repo init
     ```
     Use your IDrive username and password (or SSH key if configured).
  3. Backup your data:
     ```bash
     restic -r sftp:user@backup.idrive.com:/path/to/repo backup /your/data
     ```
  4. Migrate from R2: Export your R2 repo (`restic backup --copy`), then import to the new SFTP repo.
- **Speed**: SFTP isn’t as fast as S3—expect 20-100 MB/s depending on your connection vs. R2’s 50-200 MB/s. For incremental backups (restic’s strength), this is fine—small diffs upload quickly (e.g., 100 MB in 1-5s). Latency is 50-200ms, similar to R2’s uncached.
- **Cost**: 5 TB on R2 is $900/year + ops (~$906). IDrive’s $70/year (or $35 first year) saves you $871-$836. Even at 1 TB, $180 (R2) vs. $14 (IDrive) is a $166 win.

**Verdict**: Yes, restic works with regular IDrive via SFTP. Slower than R2, but the $7-$14/TB/year cost trumps speed for backups.

#### IDrive e2 for AppFlowy and Immich
- **Compatibility**: Already S3-compatible, so AppFlowy and Immich integrate natively. Configure e2’s endpoint (e.g., `us-east-1.idrivee2-23.com`), access key, and secret key in each app’s settings or Docker env vars.
- **Setup**:
  - AppFlowy: Edit `S3_URL`, `S3_ACCESS_KEY`, etc., in `appflowy_cloud/.env`.
  - Immich: Set `IMMICH_EXTERNAL_STORAGE=S3`, `S3_ENDPOINT`, etc., in `docker-compose.yml`.
  - Mount on Linux (optional) with `s3fs` as discussed earlier.
- **Speed**: 100-500 MB/s near region (0.04-0.2s per 20 MB Immich upload, 0.1-0.2s AppFlowy doc fetch at 50-100ms). Beats R2’s 50-200 MB/s uploads, lags R2’s 20-50ms cached reads.
- **Cost**: 1 TB at $24.75 first year vs. R2’s $186—$161 savings. 5 TB at $123.75 vs. $906—$782 savings.

**Verdict**: e2’s a slam dunk for AppFlowy and Immich—faster uploads than R2, S3-ready, and way cheaper.

### Comparing to R2 (Your Current Setup)
- **Restic on R2**: 50-200 MB/s uploads, 20-800ms reads (cached/uncached), $180/TB/year + ops. Great speed, high cost.
- **Restic on IDrive**: 20-100 MB/s via SFTP, 50-200ms latency, $14/TB/year. Slower but dirt cheap for backups.
- **AppFlowy/Immich on e2**: 100-500 MB/s uploads, 50-300ms reads, $24.75/TB first year. Outpaces R2 uploads, lags on global reads, saves big.
- **AppFlowy/Immich on R2**: 50-200 MB/s uploads, 20-800ms reads, $186/TB. Global read edge, pricier.

### Your New Setup
- **Restic to IDrive ($70/5TB)**: Handles your backup needs (e.g., server snapshots, files) with 5 TB capacity. SFTP is slower (20-100 MB/s), but restic’s incremental nature minimizes this—daily diffs upload in seconds. Saves you $836-$871 over R2’s 5 TB cost.
- **e2 for AppFlowy/Immich ($24.75/TB first year)**: Perfect S3 backend—fast uploads for Immich (100-500 MB/s), solid regional reads for AppFlowy (50-100ms). Multi-region buckets optimize per user location, all under one account.

### Practical Notes
- **Migration**: Use `rclone` to move R2 data to IDrive SFTP (`rclone sync s3:r2-bucket sftp:idrive:/path`) and e2 (`rclone sync s3:r2-bucket s3:e2-bucket`). Test restic restores after.
- **Region**: Pick e2 regions near your server/users (e.g., Virginia for US East) for max speed. IDrive SFTP is US-based, so US servers get the best 20-100 MB/s.
- **Capacity**: 5 TB IDrive covers backups; scale e2 as needed (e.g., 1 TB at $24.75 starts you off).

### Final Take
Your plan rocks:
- **IDrive ($70/5TB)** for restic: Cheap, reliable, SFTP works—saves you hundreds over R2.
- **e2 ($24.75/TB first year)** for AppFlowy/Immich: S3-compatible, faster uploads than R2, unbeatable price.

Where’s your server/users based? That’ll fine-tune e2’s region choice for peak speed!