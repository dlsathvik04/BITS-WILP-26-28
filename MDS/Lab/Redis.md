# Redis CLI Quick Syntax Reference

A fast, syntax-first study guide covering all core Redis data structures (Strings, Hashes, Lists, Sets, Sorted Sets), key expiration, and common operational patterns.

---

## 1. Connectivity & Server Check

```
# Ping server (returns PONG)
PING
```

---

## 2. Strings (Key-Value & Counters)

### 2.1 Basic Storage & Retrieval
```
# Set a single key-value pair
SET name "Ashish"

# Get value by key
GET name

# Set multiple keys in one command
MSET course "Modern Databases" faculty "Ashish" semester "Jul2026"

# Get multiple keys in one command
MGET course faculty semester

# Append string to existing value
APPEND city " Delhi"

# Get string length
STRLEN city
```

### 2.2 Key Lifecycle & Checks
```
# Check if key exists (returns 1 or 0)
EXISTS city

# Delete key
DEL city

# Set key only if it does NOT exist (returns 1 if set, 0 if exists)
SETNX lock:course "locked"
```

### 2.3 Atomic Integer Counters
```
SET visitors 100

# Increment by 1
INCR visitors

# Increment by custom step
INCRBY visitors 10

# Decrement by 1
DECR visitors

# Decrement by custom step
DECRBY visitors 20
```

---

## 3. TTL & Key Expiration (Caching)

```
# Set key with TTL in seconds
SET otp "458923" EX 60

# Set key with TTL only if key does not exist
SET cache:course:MD5 "Modern Database Systems" NX EX 300

# Add/refresh expiration on an existing key
EXPIRE cache:course:MD5 300

# Check remaining time-to-live in seconds
# Returns >0: remaining seconds | -1: no expiration | -2: expired/does not exist
TTL otp

# Remove expiration (make key permanent)
PERSIST otp
```

---

## 4. Hashes (Object Records / Key-Value Pairs)

### 4.1 Write & Read Fields
```
# Set one or more fields in a hash
HSET student:101 name "Ashish" age 35 department "CSIS"

# Get single field
HGET student:101 name

# Get multiple fields
HMGET student:101 name age department

# Get all fields and values
HGETALL student:101
```

### 4.2 Inspection, Updates & Deletion
```
# List all field keys
HKEYS student:101

# List all field values
HVALS student:101

# Count total fields in hash
HLEN student:101

# Check string length of field value
HSTRLEN student:101 name

# Check if field exists (1 or 0)
HEXISTS student:101 age

# Delete field from hash
HDEL student:101 age

# Increment numeric field
HINCRBY student:101 marks 5

# Set field only if it does NOT already exist
HSETNX student:101 email "ashish@bits.edu"

# Incremental scan over hash fields
HSCAN student:101 0
```

---

## 5. Lists (Queues, Stacks & Sequences)

### 5.1 Push & Read
```
# Prepend (push to head/left)
LPUSH fruits Orange Mango Apple

# Append (push to tail/right)
RPUSH fruits Banana Grapes

# Read elements in range (0 to -1 reads all)
LRANGE fruits 0 -1

# Count elements in list
LLEN fruits
```

### 5.2 Pop Operations
```
# Remove and return leftmost element
LPOP fruits

# Remove and return rightmost element
RPOP fruits
```

### 5.3 Modify, Insert & Trim
```
# Read element by index (0-based)
LINDEX fruits 1

# Overwrite element at specific index
LSET fruits 1 "Kiwi"

# Remove occurrences (count=1: first match; count=0: all matches)
LREM fruits 1 "Apple"

# Insert relative to a pivot element
LINSERT fruits BEFORE Banana Pear
LINSERT fruits AFTER Banana Cherry

# Trim list to retain only specified range
LTRIM fruits 0 2
```

### 5.4 Transfer & Blocking Pops
```
# Move element from one list to another (e.g. Right to Left)
LMOVE fruits basket RIGHT LEFT

# Legacy shortcut: pop right from src, push left to dest
RPOPLPUSH fruits basket

# Blocking pop (wait up to timeout seconds if queue is empty)
BLPOP queue 30
BRPOP queue 30
BLMOVE queue processing LEFT RIGHT 30
```

---

## 6. Sets (Unordered Unique Collections)

### 6.1 Basic Set Operations
```
# Add unique members (duplicates ignored)
SADD course:MD5 ST01 ST02 ST03

# Get all members (unordered)
SMEMBERS course:MD5

# Count members
SCARD course:MD5

# Check membership (returns 1 or 0)
SISMEMBER course:MD5 ST02
```

### 6.2 Removal & Transfer
```
# Remove specific member
SREM course:MD5 ST03

# Remove and return a random member
SPOP course:MD5

# Return random member without removing it
SRANDMEMBER course:MD5

# Move member between sets
SMOVE course:MD5 course:AI ST02

# Incremental scan over set members
SSCAN course:MD5 0
```

### 6.3 Set Algebra (Compare & Store)
```
# Union: all unique elements across sets
SUNION course:MD5 course:AI

# Intersection: common elements in both sets
SINTER course:MD5 course:AI

# Difference: elements in first set but not second
SDIFF course:MD5 course:AI

# Store results directly in a new set
SUNIONSTORE course:All course:MD5 course:AI
SINTERSTORE course:Common course:MD5 course:AI
SDIFFSTORE course:MD5Only course:MD5 course:AI
```

---

## 7. Sorted Sets (Leaderboards & Scoring)

```
# Add member(s) with numeric score: ZADD key score member
ZADD course:MDS:leaderboard 86 st01 92 st02 78 st03 95 st04

# Read range in descending order (highest score first)
ZRANGE course:MDS:leaderboard 0 -1 REV WITHSCORES

# Increment a member's score
ZINCRBY course:MDS:leaderboard 8 st01

# Get score of a member
ZSCORE course:MDS:leaderboard st01

# Get rank sorted high-to-low (0-based index)
ZREVRANK course:MDS:leaderboard st01
```

---

## 8. Common Patterns (Cheat Sheet)

### FIFO Queue
```
RPUSH job_queue "task1" "task2"   # Enqueue
LPOP job_queue                    # Dequeue
```

### Fixed-Window Rate Limiter (e.g., 3 requests / 20 seconds)
```
INCR rate_limit:st01              # Increment count
EXPIRE rate_limit:st01 20         # Set window expiration on 1st request
# Allow if count <= 3, reject if count > 3
```

### Atomic Counter
```
INCR page:views                   # Thread-safe counter increment
```