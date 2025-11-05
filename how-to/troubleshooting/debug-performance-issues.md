# How to Debug Performance Issues

> **Document Type**: How-To Guide (Problem-Oriented)  
> **Category**: Troubleshooting  
> **Difficulty**: Advanced
> **Last Updated**: 01/20/2025§

## Problem Statement

Identify and resolve application performance bottlenecks.

---

## Prerequisites

- [ ] Access to monitoring tools
- [ ] Basic understanding of profiling

---

## Solution

### Step 1: Identify the Bottleneck

Use profiling tools:

```bash
npm run profile
```

Look for functions consuming most CPU time.

---

### Step 2: Analyze Database Queries

Enable slow query log:

```sql
SET slow_query_log = 1;
SET long_query_time = 2;
```

Review slow queries in logs.

---

### Step 3: Optimize Critical Paths

Add caching:

```javascript
const cache = new Cache();
const result = cache.get(key) || expensiveOperation();
```

---

### Step 4: Measure Improvements

```bash
npm run benchmark
```

Compare before/after metrics.

---

## Verification

- [ ] Response times improved
- [ ] CPU usage reduced
- [ ] No new errors introduced

---

## Common Causes

- Unoptimized database queries
- Missing indexes
- Excessive API calls
- Memory leaks

---

**Related**: [Performance Best Practices](../../explanation/concepts/performance-optimization.md)
