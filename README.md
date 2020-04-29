# Interactive SQL interview on Google Colaboratory

Recently I had a technical interview on SQL performed over Webex, with tasks and solutions typed into a shared Google Doc.  This is quite standard and works fine.  But wouldn't it be nice if all participants of the process could interact with a real database, more closely approximating real work?

This solution uses Google Colaboratory, where a SQLite database is created at runtime using Python Pandas, and the SQL queries are written into templated solution cells.

## Sample task
We have a table `events` that looks like this:
```
events (
  account STRING,
  event STRING,
  timestamp DATETIME
)
```
Provide counts of each event type for each account.
```
output (
  account STRING,
  event1_count INT,
  event2_count INT,
  event3_count INT
)
```

## Create a database with sample data
```
import numpy as np
import pandas as pd
import secrets
import sqlite3
import time

accounts = [secrets.token_hex(2) for i in range(5)]
event_types = ['PLAY','STOP','ERROR']
time_now = int(time.time()*1e3)
N = 100
df = pd.DataFrame({
    'account': np.random.choice(accounts, size=N),
    'event': np.random.choice(event_types, p=[.5, .3, .2], size=N),
    'timestamp': [(time_now) - np.random.randint(0, 1e6) for i in range(N)],
}).sort_values(['timestamp'])
conn = sqlite3.connect('interview.sqlite')
df.to_sql('events', conn, if_exists='replace', index=False)
```

## Solution
```
# solution - using CASE because SQLite does not have IF
q = '''
SELECT
  account
  ,SUM(CASE event WHEN 'PLAY' THEN 1 ELSE 0 END)  AS count_PLAY
  ,SUM(CASE event WHEN 'STOP' THEN 1 ELSE 0 END)  AS count_STOP
  ,SUM(CASE event WHEN 'ERROR' THEN 1 ELSE 0 END) AS count_ERROR
FROM events
GROUP BY account
'''
result = pd.read_sql(q, conn)
result
```

## Output
| | account | count_PLAY | count_STOP | count_ERROR |
|-|---------|------------|------------|-------------|
|0| 251a    | 12         | 5          | 3           |
|1| e4b0    | 14         | 3          | 2           |
|2| 78a3    | 15         | 2          | 4           |
|3| ba75    | 10         | 5          | 2           |
|4| 9c11    | 17         | 2          | 3           |
