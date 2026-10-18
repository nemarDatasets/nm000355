# nm000355 stimulus-level events

Within-epoch event rows of the epoched recordings of nm000355, moved here on 18 Oct 2026 from the raw `events.tsv`, which now holds its `epoch` rows (one per epoch, already present, unchanged; the `value` column, n/a on every one of them, was dropped) so that `mne_bids.read_epochs_bids` loads each file as its epochs.

- 1 files, 26 rows; same file names, columns, line text and order as the raw files before the move; `onset`/`sample` are relative to the start of the recording.
- Rows identical in every column to an earlier row were dropped (0 rows).
- 0 rows lie outside the recording (kept as delivered).
- `stimulus_events_summary.tsv`: per raw file, epochs, rows before, duplicates dropped, rows here.
