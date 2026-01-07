# Database Practice

## How to Use

### Start all databases

```bash
make up
```

### Connect to databases

```bash
make psql     # PostgreSQL
make mongo    # MongoDB shell
make redis    # Redis CLI
```

### Stop everything

```bash
make down
```

### Full cleanup (⚠ deletes data)

```bash
make clean
```

---

## Connection Info (for apps / clients)

### PostgreSQL

```
host: localhost
port: 5432
user: devuser
password: devpass
database: devdb
```

### MongoDB

```
mongodb://localhost:27017
```

### Redis

```
redis://localhost:6380
```

---

