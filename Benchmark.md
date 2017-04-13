# Running the benchmark

To run the benchmark yourself:
```bash
git clone https://github.com/JoshuaWise/better-sqlite3.git
cd better-sqlite3
npm install # if you're doing this as the root user, --unsafe-perm is required
node benchmark
```

# Results

These results are from 04/13/2017, on a MacBook Pro (Retina, 15-inch, Mid 2014, OSX 10.11.4), using nodejs v6.9.1.

### Reading rows individually (`.get()`)
```
better-sqlite3 x 138,926 ops/sec ±0.81%
node-sqlite3   x 17,738 ops/sec ±0.49%
```

### Reading 100 rows into an array (`.all()`)
```
better-sqlite3 x 5,871 ops/sec ±2.60%
node-sqlite3   x 2,189 ops/sec ±2.05%
```

### Iterating over 100 rows (`.each()`)
```
better-sqlite3 x 6,303 ops/sec ±1.95%
node-sqlite3   x 2,067 ops/sec ±1.82%
```

### Inserting rows individually (`.run()`)
```
--- normal journal mode ---
better-sqlite3 x 1,013 ops/sec ±14.63% (41 runs sampled)
node-sqlite3   x 788 ops/sec ±21.24% (47 runs sampled)

--- WAL mode ---
better-sqlite3 x 61,206 ops/sec ±3.05%
node-sqlite3   x 19,364 ops/sec ±1.86%
```

### Inserting 100 rows in a single transaction
```
better-sqlite3 x 1,324 ops/sec ±3.01%
node-sqlite3   x 212 ops/sec ±0.63%
```

### "Real world" benchmark
```
--- normal journal mode ---
better-sqlite3 x 4,665 ops/sec ±1.40%
node-sqlite3   x 3,412 ops/sec ±2.85%

--- WAL mode ---
better-sqlite3 x 99,606 ops/sec ±1.75%
node-sqlite3   x 17,643 ops/sec ±4.12%
```