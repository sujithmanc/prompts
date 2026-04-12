### Key Improvements in `xdb`:
* **Storage Efficiency:** Switched `date` from `VARCHAR(10)` to `DATE` (saves 8 bytes per row).
* **Referential Integrity:** `ON DELETE CASCADE` means if you delete a topic, the mess is cleaned up automatically.
* **Surgical Precision:** The `idx_notes_topic_date` composite index means your `EXPLAIN` plan will now show `rows: 1` or very low numbers consistently.
