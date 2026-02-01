# MVP 해커톤 DB 스키마 (SQLite3)

본 문서는 **SQLite3 기준**으로 작성된 DB 스키마 초안이다.  
데이터 타입은 SQLite 권장 타입(`INTEGER`, `TEXT`, `REAL`, `BLOB`)을 사용한다.

---

## 1. users

```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    role TEXT NOT NULL CHECK (role IN ('MENTOR','MENTEE')),
    name TEXT NOT NULL,
    created_at TEXT NOT NULL
);
```
## 2. Subjects
```sql
CREATE TABLE subjects (
    id INTERGER PRIMARY KEY AUTOINCREMENT,
    subject TEXT NOT NULL
)

---

## 3. daily_planner

```sql
CREATE TABLE daily_planner (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    mentee_id INTEGER NOT NULL,
    date TEXT NOT NULL,
    header_note TEXT,
    created_at TEXT NOT NULL,
    updated_at TEXT NOT NULL,
    UNIQUE (mentee_id, date),
    FOREIGN KEY (mentee_id) REFERENCES users(id)
);
```

---

## 4. tasks

```sql
CREATE TABLE tasks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    mentee_id INTEGER NOT NULL,
    date TEXT NOT NULL,
    time TEXT NOT NULL,
    subject TEXT NOT NULL CHECK (subject IN ('KOR','ENG','MATH','ETC')),
    title TEXT NOT NULL,
    goal TEXT,
    is_fixed_by_mentor INTEGER NOT NULL CHECK (is_fixed_by_mentor IN (0,1)),
    created_by INTEGER NOT NULL,
    status TEXT NOT NULL CHECK (status IN ('TODO','WORKING', 'DONE')),
    completed_at TEXT,
    sort_order INTEGER,
    created_at TEXT NOT NULL,
    updated_at TEXT NOT NULL,
    FOREIGN KEY (mentee_id) REFERENCES users(id),
    FOREIGN KEY (created_by) REFERENCES users(id)
);
```

---

## 5. task_time_logs

```sql
CREATE TABLE task_time_logs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    task_id INTEGER NOT NULL,
    started_at TEXT NOT NULL,
    ended_at TEXT,
    duration_seconds INTEGER,
    source TEXT CHECK (source IN ('manual','timer')),
    created_at TEXT NOT NULL,
    FOREIGN KEY (task_id) REFERENCES tasks(id)
);
```

---

## 6. task_materials

```sql
CREATE TABLE task_materials (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    task_id INTEGER NOT NULL,
    type TEXT NOT NULL CHECK (type IN ('COLUMN','PDF')),
    title TEXT,
    content TEXT,
    file_url TEXT,
    uploaded_by INTEGER NOT NULL,
    created_at TEXT NOT NULL,
    FOREIGN KEY (task_id) REFERENCES tasks(id),
    FOREIGN KEY (uploaded_by) REFERENCES users(id)
);
```

---

## 7. task_submissions

```sql
CREATE TABLE task_submissions (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    task_id INTEGER NOT NULL,
    mentee_id INTEGER NOT NULL,
    image_url TEXT NOT NULL,
    note TEXT,
    submitted_at TEXT NOT NULL,
    FOREIGN KEY (task_id) REFERENCES tasks(id),
    FOREIGN KEY (mentee_id) REFERENCES users(id)
);
```

---

## 8. feedbacks

```sql
CREATE TABLE feedbacks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    mentee_id INTEGER NOT NULL,
    mentor_id INTEGER NOT NULL,
    date TEXT NOT NULL,
    subject TEXT NOT NULL CHECK (subject IN ('KOR','ENG','MATH','ETC')),
    summary TEXT,
    body TEXT,
    overall_comment TEXT,
    created_at TEXT NOT NULL,
    updated_at TEXT NOT NULL,
    UNIQUE (mentee_id, date, subject),
    FOREIGN KEY (mentee_id) REFERENCES users(id),
    FOREIGN KEY (mentor_id) REFERENCES users(id)
);
```

---

## 9. calendar_events

```sql
CREATE TABLE calendar_events (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    mentee_id INTEGER NOT NULL,
    title TEXT NOT NULL,
    start_date TEXT NOT NULL,
    end_date TEXT,
    subject TEXT CHECK (subject IN ('KOR','ENG','MATH','ETC')),
    description TEXT,
    created_by INTEGER NOT NULL,
    created_at TEXT NOT NULL,
    updated_at TEXT NOT NULL,
    FOREIGN KEY (mentee_id) REFERENCES users(id),
    FOREIGN KEY (created_by) REFERENCES users(id)
);
```

---

## 10. notifications

```sql
CREATE TABLE notifications (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL,
    type TEXT NOT NULL,
    title TEXT NOT NULL,
    body TEXT NOT NULL,
    payload_json TEXT,
    is_read INTEGER NOT NULL CHECK (is_read IN (0,1)),
    sent_at TEXT,
    created_at TEXT NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

---

### 참고
- 시간/날짜는 **ISO-8601 문자열(TEXT)** 로 저장
- SQLite에서는 BOOLEAN → INTEGER(0/1)
- MVP 이후 Postgres/MySQL로 이전 가능하도록 설계
