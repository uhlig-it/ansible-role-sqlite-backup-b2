# SQLite Backup to a B2 Bucket

Performs scheduled backups of an SQLite database and retains them.

A systemd timer creates a daily backup of the database (based on this [article](https://litestream.io/alternatives/cron/)). An S3 bucket is set up to keep the backup files. It is set without versioning, so that we can retain generations independently of the bucket settings.

# Ansible Variables

* `sqlite_backup_b2_database_path` - path to the database file to backup; _required_; e.g. `{{ ansistrano_shared_path }}/production.sqlite3`
* `sqlite_backup_b2_bucket` - name of the B2 bucket to store the backups in; _required_
* `sqlite_backup_b2_keyId` - `keyId` for the b2 bucket; _required_
* `sqlite_backup_b2_applicationKey` - `applicationKey` for the b2 bucket; _required_
* `sqlite_backup_b2_prefix` - common prefix for backup files; _optional_; defaults to `ansible_play_name`
* `sqlite_backup_b2_runtime_user` - runtime user to perform backup
* `sqlite_backup_b2_runtime_group` - group of the runtime user to perform backup

# Retention Strategy

* Daily backup; kept for seven days
* Sunday backup is kept forever (TODO for `n` weeks)
* TODO Backup of the last day of the month is kept for `n` months
* Backup of the last day of the year is kept forever (TODO for `n` years)

As a result, there will be seven daily backup files in the S3 bucket:

- `something.Monday.sqlite3`
- `something.Tuesday.sqlite3`
- ...
- `something.Sunday.sqlite3`

Likewise, there is a backup file for each week of the year, copied on Sunday:

- `something.week-01.sqlite3`
- ...
- `something.week-53.sqlite3`

Same for each year; the file of Dec 31st of each year is kept as:

- `something.2022.sqlite3`
- `something.2023.sqlite3`
- ...
