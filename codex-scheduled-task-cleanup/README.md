# Codex Scheduled Task Cleanup

A reusable ChatGPT Skill for diagnosing and cleaning up **stale historical Codex scheduled-task run threads** that remain visible in the macOS ChatGPT/Codex project sidebar.

## The problem

A single recurring Codex automation can legitimately create a new local session each time it runs. Over time, the project sidebar may show many rows with the same title even though **only one recurring schedule exists**.

In the failure mode this project targets, archiving the old session file alone is not enough: the ChatGPT/Codex Desktop app may still retain stale automation-run and sidebar-catalog state.

## What this project does

It separates three layers:

1. the recurring automation definition;
2. each historical run thread/session;
3. Desktop-local automation/sidebar state.

The included cleanup script:

- defaults to dry-run;
- matches one exact thread title;
- keeps the newest run by default;
- backs up SQLite state before writes;
- archives stale sessions with the Codex CLI;
- reconciles stale `automation_runs` and `local_thread_catalog` rows;
- verifies the newest run remains active/visible;
- is idempotent.

## Quick start

Dry run:

```bash
python3 scripts/cleanup_codex_scheduled_runs.py \
  --title "My scheduled task" \
  --automation-id "my-automation-id"
```

Apply:

```bash
python3 scripts/cleanup_codex_scheduled_runs.py \
  --title "My scheduled task" \
  --automation-id "my-automation-id" \
  --apply
```

Then fully quit and reopen ChatGPT Desktop.

Optional automatic cleanup after each scheduled run:

```bash
python3 scripts/install_launchagent.py \
  --cleanup-script "$(pwd)/scripts/cleanup_codex_scheduled_runs.py" \
  --title "My scheduled task" \
  --automation-id "my-automation-id" \
  --times "06:10,11:10,16:10"
```

## Safety

This project intentionally does **not** permanently delete chats or schedules. It uses exact-title matching, keeps at least one run, creates database backups, and aborts if expected schemas are missing.

The local Codex database structures used here are implementation details and can change across releases. Always run dry-run first after a Codex update.

## Install as a ChatGPT Skill

Upload the packaged `skill.zip` to ChatGPT Skills, or use the repository folder as the source when building/installing a personal Skill.

## License

MIT. See `LICENSE`.
