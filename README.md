# presql
PostgreSQL and Psycopg2 wrapper.

# Official Release

PreSQL can now be used on your Python projects through PyPi by running pip command on a Python-ready environment.

`pip install presql --upgrade`

Current version is 1.0.0, but more updates are coming soon. Installing it will also install required packages including `psycopg2`.

This is compatible with Python 3.9 or with the latest version.

### package import
`from presql import PreSQL`

There are many ways to connect to a Postgres server, check Psycopg2 [Connection](https://www.psycopg.org/docs/connection.html) documentation for more details. Some of those features may or may not be implement as of the current version.

**Important!** As always, do not store your credentials in the project folder.

### Connect using a valid Database URI
```python
with PreSQL(DATABASE_URI, sslmode="require") as db:
    print("\nConnect via URI", db.connected())
```

### Connect using a valid DSN String
```python
with PreSQL(f"dbname={DATABASE} user={USER} password={PASSWORD}") as db:
    print("\nConnect via DSN", db.connected())
``` 

### Connect using parameters
```python
with PreSQL(dbname=DATABASE, user=USER, password=PASSWORD, host=HOST, port=PORT) as db:
    print("\nConnect using parameters", db.connected())
```

### Manual SQL commands
```python
with PreSQL(dbname=DATABASE, user=USER, password=PASSWORD, host=HOST, port=PORT) as db:
    if db.connected():
        query = """
            CREATE TABLE IF NOT EXISTS inventory (
                inventory_id integer GENERATED ALWAYS AS IDENTITY NOT NULL,
                item varchar,
                total int,
                modified int); """
        db.execute(query)
```

### Manual SQL commands
```python
with PreSQL(dbname=dbname, user=user, password=password, host=host, port=port) as db:
    if db.connected():
        rows = db.execute("SELECT * FROM inventory LIMIT 5;").fetchall()
        for row in rows:
            print(" -", row["item"])
```

### COUNT shortcut
```python
with PreSQL(dbname=DATABASE, user=USER, password=PASSWORD, host=HOST, port=PORT) as db:
    if db.connected():
        count = db.count("inventory", where="item='ramen'")
        print("Count:", count)
```

### SELECT shortcut
```python
with PreSQL(dbname=DATABASE, user=USER, password=PASSWORD, host=HOST, port=PORT) as db:
    if db.connected():
        rows = db.select("inventory", "total", where="item='ramen'")
        for row in rows:
            print(row["total"])
```

### INSERT Shortcut
```python
with PreSQL(dbname=dbname, user=user, password=password, host=host, port=port) as db:
    if db.connected():
        count = db.count("tokens", where="token='ethereum'")
        if not count:
            db.insert("tokens", columns="token, usage, modified", values=f"('ethereum', 0, {modified})")
```

### Auto-Commit = False, Rollback
```python
with PreSQL(dbname=DATABASE, user=USER, password=PASSWORD, host=HOST, port=PORT, autocommit=False) as db:
    if db.connected():
        db.insert("inventory", columns="item, usage, modified", values=f"('jelly', 0, {modified})")
        # some failed transactions
        db.rollback()
```

### UPDATE shortcut
```python
with PreSQL(dbname=DATABASE, user=USER, password=PASSWORD, host=HOST, port=PORT) as db:
    if db.connected():
        total = 0
        rows = db.select("inventory", "total", where="item='ramen'")
        for row in rows:
            total = int(row["total"]) + 1
        db.update("inventory", columns=f"total={total}", where="item='ramen'")
        print("Usage:", total)
```

### DELETE shortcut
```python
with PreSQL(dbname=DATABASE, user=USER, password=PASSWORD, host=HOST, port=PORT) as db:
    if db.connected():
        db.insert("inventory", columns="item, total, modified", values=f"('soy milk', 0, {modified})")
        count = db.count("inventory", where="item='soy milk'")
        print("Anon:", count)
        
        db.delete("inventory", where="item='soy milk'")
        count = db.count("inventory", where="item='soy milk'")
        print("Anon:", count)
```