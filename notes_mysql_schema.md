### 1. Database Creation
```sql
CREATE DATABASE IF NOT EXISTS xdb;
USE xdb;
```

### 2. Table Structures
We are using `DATE` for the calendar day and `TIMESTAMP` for the exact creation/update moments.

```sql
-- 1. Topics Table
CREATE TABLE topics (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 2. Subtopics Table
CREATE TABLE subtopics (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    topic_id BIGINT UNSIGNED NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_subtopics_topic FOREIGN KEY (topic_id) 
        REFERENCES topics(id) ON DELETE CASCADE
);

-- 3. Notes Table (qa_notes)
CREATE TABLE qa_notes (
    id SERIAL PRIMARY KEY,
    que TEXT NOT NULL,
    ans TEXT NOT NULL,
    topic_id BIGINT UNSIGNED NOT NULL,
    subtopic_id BIGINT UNSIGNED,
    note_date DATE NOT NULL, -- Proper DATE type
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    -- Foreign Key Constraints
    CONSTRAINT fk_notes_topic FOREIGN KEY (topic_id) 
        REFERENCES topics(id) ON DELETE CASCADE,
    CONSTRAINT fk_notes_subtopic FOREIGN KEY (subtopic_id) 
        REFERENCES subtopics(id) ON DELETE SET NULL
);
```

---

### 3. Performance Optimization (Indexes)
Based on your usage patterns (filtering by date and topics), these indexes ensure $O(\log n)$ search speeds.

```sql
-- Composite index for your most frequent query pattern (Topic + Date)
CREATE INDEX idx_notes_topic_date ON qa_notes(topic_id, note_date);

-- Index for searching by date alone (Daily study reviews)
CREATE INDEX idx_notes_date ON qa_notes(note_date);

-- Index for subtopic lookups
CREATE INDEX idx_notes_subtopic ON qa_notes(subtopic_id);
```



---

### 4. The Simplified View
This view allows you to query by name (e.g., "Bible") while the database handles the heavy lifting via IDs in the background.

```sql
CREATE OR REPLACE VIEW v_notes AS
SELECT 
    n.id AS note_id,
    t.name AS topic_name,
    s.name AS subtopic_name,
    n.que,
    n.ans,
    n.note_date,
    n.created_at,
    -- Including IDs in the view allows for even faster filtering if needed
    n.topic_id,
    n.subtopic_id
FROM qa_notes n
JOIN topics t ON n.topic_id = t.id
LEFT JOIN subtopics s ON n.subtopic_id = s.id;
```

---

### 5. How to use this new Clean DB
Now, your queries are simple, clean, and incredibly fast:

**Search by Topic and Date:**
```sql
SELECT * FROM v_notes 
WHERE topic_name = 'Bible' 
AND note_date = '2026-04-12';
```

**Search for a Range of Dates:**
```sql
SELECT * FROM v_notes 
WHERE note_date BETWEEN '2026-04-01' AND '2026-04-30';
```
