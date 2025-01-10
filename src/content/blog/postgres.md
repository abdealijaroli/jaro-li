---
title: PostgreSQL - Table Inheritance and Partition Pruning
description: At work, we recently implemented partitioning on our new PostgreSQL tables, and I would love to highlight Postgres' absolute beauty.
date: 2025-01-10
# lang: en
duration: 8min
# tag: hello
# redirect: /
# video: true
---

At work, we recently implemented partitioning on our new PostgreSQL tables, and I couldn’t help but admire Postgres’ elegance in handling scalability.

---

## Partitioning?

Partitioning is the process of splitting a large table into smaller, manageable parts (called **partitions**) based on specific criteria, such as date ranges or IDs. Each partition acts as an independent table while logically remaining part of the main table.

**Key Benefits:**

-   **Improved Query Performance**: Queries only scan relevant partitions, skipping unrelated data.
-   **Smaller Indexes**: Each partition has its own index, reducing search time.
-   **Simplified Maintenance**: Dropping old or irrelevant data becomes effortless by simply removing the associated partition, avoiding row-by-row deletions.

---

## How PostgreSQL Handles Partitioning

Here's where the juice is--PostgreSQL's **declarative partitioning** makes partitioning seamless. How it works:

### 1. **Partitioned Tables and Child Tables**

A **partitioned table** is a logical parent table, while its data resides in **child tables** (partitions). For example:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT,
    created_at DATE
) PARTITION BY RANGE (id);

CREATE TABLE users_partition_1
    PARTITION OF users
    FOR VALUES FROM (1) TO (1000);

CREATE TABLE users_partition_2
    PARTITION OF users
    FOR VALUES FROM (1001) TO (2000);
```

### 2. **Automatic Data Routing**

When inserting data, PostgreSQL automatically routes rows to the correct partition based on the defined criteria:

```sql
INSERT INTO users (id, name, created_at) VALUES (1500, 'Uday Shawty', '2025-01-01');
```

Here, the row is stored in `users_partition_2` because `id=1500` falls within its range.

### 3. **Partition Pruning**

During a query, PostgreSQL intelligently skips irrelevant partitions:

```sql
SELECT * FROM users WHERE id = 500;
```

Only `users_partition_1` is scanned, significantly improving query performance.

---

## Partitioning Strategies

PostgreSQL supports multiple partitioning methods:

1. **Range Partitioning**:

    - Splits data into ranges (e.g., dates, IDs).
    - Example: `FOR VALUES FROM ('2024-01-01') TO ('2025-01-01')`.

2. **List Partitioning**:

    - Divides data based on discrete values (e.g., regions).
    - Example: `FOR VALUES IN ('North', 'South')`.

3. **Hash Partitioning**:
    - Uses a hash function to evenly distribute data.
    - Example: `PARTITION BY HASH (id)`.

---

## Performance Benefits of Partitioning

### **Faster Queries**

Partition pruning ensures that only relevant partitions are scanned, reducing search time.

### **Efficient Indexing**

Each partition maintains its own index, keeping searches within manageable bounds.

### **Easy Data Management**

Dropping outdated data (e.g., logs) is straightforward. Instead of deleting rows, you can simply drop the partition:

```sql
DROP TABLE users_partition_1;
```

---

## Practical Example: Query Optimization

Suppose you have a `transactions` table partitioned by year:

```sql
CREATE TABLE transactions (
    id SERIAL,
    amount NUMERIC,
    transaction_date DATE
) PARTITION BY RANGE (transaction_date);

CREATE TABLE transactions_2024
    PARTITION OF transactions
    FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');
```

Query:

```sql
SELECT * FROM transactions WHERE transaction_date = '2024-06-15';
```

**What Happens:**

1. PostgreSQL identifies `transactions_2024` as the relevant partition.
2. Other partitions are ignored, reducing scan time.

---

## Challenges and Considerations

1. **Partition Management**:

    - Adding or dropping partitions requires careful planning.

2. **Data Skew**:

    - Uneven distribution of data (e.g., most users having IDs in a single range) can lead to overloaded partitions, reducing the benefits of partitioning.

3. **Complex Queries**:
    - Joins across partitions can be less efficient.

---

## Conclusion

Partitioning in PostgreSQL was a turning point in scaling our database at work. It’s a game-changer for managing large datasets. By splitting tables into smaller partitions, we achieved faster queries, reduced index sizes, and simplified maintenance. Whether you're working on time-series data or large-scale user datasets, PostgreSQL works wonders.
