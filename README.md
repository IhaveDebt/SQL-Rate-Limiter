# rate_limiter.py
import psycopg2
from datetime import datetime, timedelta

DB_PARAMS = {
    "dbname": "ratelimit_db",
    "user": "postgres",
    "password": "password",
    "host": "localhost",
    "port": 5432
}

def allow_request(user_id, limit=5, window_seconds=60):
    conn = psycopg2.connect(**DB_PARAMS)
    cur = conn.cursor()
    now = datetime.now()
    window_start = now - timedelta(seconds=window_seconds)

    cur.execute("DELETE FROM requests WHERE requested_at < %s;", (window_start,))
    cur.execute("SELECT COUNT(*) FROM requests WHERE user_id=%s;", (user_id,))
    count = cur.fetchone()[0]

    if count >= limit:
        print(f"Request denied for {user_id}")
        allowed = False
    else:
        cur.execute("INSERT INTO requests(user_id, requested_at) VALUES (%s, %s);", (user_id, now))
        conn.commit()
        print(f"Request allowed for {user_id}")
        allowed = True

    cur.close()
    conn.close()
    return allowed

if __name__ == "__main__":
    for _ in range(7):
        allow_request("user123")
