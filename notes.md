# STAR Interview Notes — CI/CD Pipeline Optimization

**Project:** SREDSimplify  
**Tech Stack:** GitHub Actions, Next.js 14, React, TypeScript, Node.js, npm, YAML  
**Jira:** SRED-2 (CI Baseline), SRED-3 (CI Optimization)  
**GitHub PR:** #79 (Baseline), #80 (Optimization)

---

## Interview Question

**Tell me about a time you improved the performance of a development process.**

---

# 1. S — Situation

In one of my projects, I worked on a full-stack application called SREDSimplify.

The frontend was built with Next.js, React, and TypeScript.

We used GitHub Actions to automatically check and build our code.

Originally, the frontend CI pipeline had four main tasks:

- ESLint
- TypeScript type checking
- Unit tests
- Production build

All these tasks ran sequentially in a single job.

The entire frontend CI job took about **123 seconds**.

### Original CI Pipeline

```text
Frontend CI Job
       |
       v
    npm ci
       |
       v
     ESLint
       |
       v
   Type Check
       |
       v
   Unit Tests
       |
       v
   Next Build
       |
       v
    Complete
```

### Original Workflow (Simplified)

```yaml
jobs:
  frontend:
    name: Frontend CI
    runs-on: ubuntu-latest

    defaults:
      run:
        working-directory: frontend

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"
          cache-dependency-path: frontend/package-lock.json

      - name: Install dependencies
        run: npm ci

      - name: ESLint
        run: npm run lint

      - name: TypeScript type check
        run: npm run typecheck

      - name: Unit tests
        run: npm run test:unit

      - name: Build
        run: npm run build
```

### Key Concept

**Sequential execution means tasks run one after another.**

Even if two tasks are independent, they cannot run at the same time when they are defined as steps in the same job.

### Interview Explanation

> Our frontend CI pipeline was slow because all tasks ran sequentially.
>
> The pipeline included linting, type checking, unit tests, and a production build.
>
> The entire frontend job took about two minutes.

---

# 2. T — Task

My goal was to reduce the frontend CI execution time without removing any tests or quality checks.

I focused on two improvements:

1. **Parallel Execution:** Run independent tasks at the same time.
2. **Build Caching:** Reuse previous Next.js build data.

I also wanted to measure the performance before and after the changes.

### Interview Explanation

> My goal was to make the CI pipeline faster without removing any quality checks.
>
> I wanted to improve parallel execution and build caching.
>
> I also wanted to measure the results to confirm that the changes actually helped.

---

# 3. A — Action

I completed the optimization in four steps.

## Step 1 — Measure the Baseline

First, I measured the original CI performance.

I used GitHub CLI to check previous workflow runs.

```powershell
gh run list --workflow "CI"
```

Then I inspected a specific workflow run.

```powershell
gh run view 38958787688
```

I also used the GitHub Actions API to measure the execution time of each step.

```powershell
$job = gh api repos/YiWang24/SREDSimplify/actions/jobs/117168354451 |
    ConvertFrom-Json
```

```powershell
$job.steps | ForEach-Object {
    [PSCustomObject]@{
        Step = $_.name
        Seconds = (
            [datetime]$_.completed_at -
            [datetime]$_.started_at
        ).TotalSeconds
    }
} | Format-Table -AutoSize
```

### Baseline Results

| Task | Execution Time |
|---|---:|
| Install dependencies | 32s |
| ESLint | 4s |
| TypeScript type check | 11s |
| Unit tests | 2s |
| Next.js Build | 60s |
| **Total frontend job** | **123s** |

I found that dependency installation and the Next.js production build were the most expensive tasks.

### Interview Explanation

> First, I measured the existing pipeline using GitHub CLI and the GitHub Actions API.
>
> I found that the frontend job took about 123 seconds.
>
> Dependency installation and the production build were the biggest bottlenecks.

---

## Step 2 — Split the Pipeline into Parallel Jobs

Next, I separated the frontend pipeline into two independent jobs.

### Job 1: Frontend Checks

- ESLint
- TypeScript type checking
- Unit tests

### Job 2: Frontend Build

- Next.js production build

Both jobs depended on the same change detection job.

However, they did not depend on each other.

This allowed GitHub Actions to run them in parallel.

### New CI Architecture

```text
              Detect Changes
                    |
          +---------+---------+
          |                   |
          v                   v
    Frontend Checks     Frontend Build
          |                   |
       npm ci               npm ci
          |                   |
       ESLint              Next Build
          |
      Type Check
          |
      Unit Tests
          |                   |
          +---------+---------+
                    |
                    v
                CI Complete
```

### GitHub Actions Configuration

```yaml
jobs:
  frontend-checks:
    name: Frontend Checks
    needs: changes
    if: needs.changes.outputs.frontend == 'true'

    runs-on: ubuntu-latest

    defaults:
      run:
        working-directory: frontend

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"
          cache-dependency-path: frontend/package-lock.json

      - name: Install dependencies
        run: npm ci

      - name: ESLint
        run: npm run lint

      - name: TypeScript
        run: npm run typecheck

      - name: Unit tests
        run: npm run test:unit

  frontend-build:
    name: Frontend Build
    needs: changes
    if: needs.changes.outputs.frontend == 'true'

    runs-on: ubuntu-latest

    defaults:
      run:
        working-directory: frontend

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"
          cache-dependency-path: frontend/package-lock.json

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build
```

### Why Does This Work?

The key configuration is:

```yaml
needs: changes
```

Both jobs depend on `changes`.

But neither job depends on the other.

Therefore, GitHub Actions can run them at the same time when runners are available.

**Important:** Each job runs on its own runner.

That means each job needs to check out the code and install dependencies separately.

### A Problem I Encountered

During testing, both frontend jobs were skipped.

The reason was that the workflow used `dorny/paths-filter` to detect frontend changes.

However, I had only modified the CI workflow file, not the frontend source code.

I fixed the issue by adding the workflow file to the frontend change filter.

```yaml
frontend:
  - 'frontend/**'
  - '.github/workflows/ci.yml'
```

After this change, modifying the CI workflow could trigger the frontend jobs.

### Interview Explanation

> Next, I split the pipeline into two parallel jobs: Frontend Checks and Frontend Build.
>
> Both jobs depended on the same change detection job, so they could run at the same time.
>
> During testing, I noticed that both jobs were skipped.
>
> I found that the path filter did not include the workflow file.
>
> I updated the filter and confirmed that both jobs ran successfully.

---

## Step 3 — Add Next.js Build Caching

After implementing parallel jobs, I measured the new pipeline.

The Frontend Build job still took about **114 seconds**.

The Next.js build step alone took **66 seconds**.

I wanted to reduce the build time further.

### The Problem

GitHub-hosted runners are temporary.

Each workflow run normally starts with a fresh environment.

Without persistent build caching:

```text
CI Run 1
   |
   v
Next.js Build
   |
   v
Generate .next/cache
   |
   v
Runner is removed

CI Run 2
   |
   v
No previous build cache
   |
   v
Build again
```

Next.js stores reusable build data in:

```text
frontend/.next/cache
```

I used `actions/cache` to save and restore this directory between workflow runs.

### Build Cache Configuration

```yaml
- name: Cache Next.js build
  uses: actions/cache@v4
  with:
    path: frontend/.next/cache
    key: ${{ runner.os }}-nextjs-${{ hashFiles('frontend/package-lock.json') }}-${{ github.sha }}
    restore-keys: |
      ${{ runner.os }}-nextjs-${{ hashFiles('frontend/package-lock.json') }}-
```

I placed the cache step before the production build.

```yaml
- name: Install dependencies
  run: npm ci

- name: Cache Next.js build
  uses: actions/cache@v4
  with:
    path: frontend/.next/cache
    key: ${{ runner.os }}-nextjs-${{ hashFiles('frontend/package-lock.json') }}-${{ github.sha }}
    restore-keys: |
      ${{ runner.os }}-nextjs-${{ hashFiles('frontend/package-lock.json') }}-

- name: Build
  run: npm run build
```

### Understanding the Cache Configuration

#### 1. `path`

```yaml
path: frontend/.next/cache
```

This tells GitHub Actions which directory to cache.

#### 2. `key`

```yaml
key: ${{ runner.os }}-nextjs-${{ hashFiles('frontend/package-lock.json') }}-${{ github.sha }}
```

The key identifies a specific cache.

It includes:

- Operating system
- Dependency lockfile hash
- Git commit SHA

#### 3. `restore-keys`

```yaml
restore-keys: |
  ${{ runner.os }}-nextjs-${{ hashFiles('frontend/package-lock.json') }}-
```

If GitHub Actions cannot find an exact cache match, it can try to restore a compatible cache using the prefix.

### npm Cache vs. Next.js Build Cache

| Cache | Purpose |
|---|---|
| npm Cache | Reuses downloaded npm packages |
| Next.js Build Cache | Reuses intermediate build data |

The existing workflow already had npm caching:

```yaml
cache: "npm"
```

But npm caching does not cache Next.js build results.

### Interview Explanation

> After that, I focused on build caching.
>
> GitHub Actions uses temporary runners, so build cache files are normally lost after each run.
>
> I used the actions/cache action to save the Next.js cache directory.
>
> This allowed later CI runs to reuse previous build data instead of rebuilding everything from scratch.

---

## Step 4 — Verify the Cache and Measure Performance

I ran the workflow twice.

### First Run: Cache Miss

The first run showed:

```text
Cache not found for input keys
```

This was expected because no Next.js build cache existed yet.

After the build finished, the logs showed:

```text
Cache saved with key: Linux-nextjs-...
```

The cache was successfully saved.

### Second Run: Cache Hit

I reran the same workflow.

```powershell
gh run rerun 38984763654
```

Then I checked the logs.

```powershell
gh run view 38984763654 --log |
    Select-String -Pattern "Linux-nextjs|Cache restored"
```

The logs showed:

```text
Cache hit for: Linux-nextjs-...
Cache restored from key: Linux-nextjs-...
```

This confirmed that GitHub Actions successfully restored the previous Next.js build cache.

### Performance Results

| Metric | Before | After Cache Hit | Reduction |
|---|---:|---:|---:|
| Frontend Build Job | 114s | 59s | 48.2% |
| Next.js Build Step | 66s | 23s | 65.2% |

The original frontend CI job took approximately 123 seconds.

After the optimization, the two frontend jobs ran in parallel.

Their observed durations were approximately:

- Frontend Checks: 60 seconds
- Frontend Build: 59 seconds

Because the jobs ran in parallel, their durations were not added together.

**Important:** These results came from individual workflow runs. They show observed improvements, not guaranteed average performance gains.

### Interview Explanation

> Finally, I verified the optimization with real CI runs.
>
> In the first run, GitHub Actions could not find a build cache, so it created one.
>
> In the second run, the cache was successfully restored.
>
> The Next.js build step dropped from 66 seconds to 23 seconds, which was about a 65% reduction in that test.
>
> The Frontend Build job also dropped from 114 seconds to 59 seconds.

---

# 4. R — Result

The optimization was successful.

I achieved three main results:

1. **Parallel execution:** Frontend checks and production builds could run at the same time.
2. **Persistent caching:** GitHub Actions could reuse Next.js build data between runs.
3. **Measurable improvement:** The observed Next.js build time decreased from 66 seconds to 23 seconds.

I also documented the performance measurements in Jira and GitHub Pull Requests.

### What I Learned

The biggest lesson I learned was to measure performance before making changes, identify the bottleneck, and verify the results after optimization.

---

# 5. Complete STAR Answer — 90-Second Interview Version

**Question: Tell me about a time you improved the performance of a development process.**

### Situation

In one of my projects, I worked on improving our CI pipeline using GitHub Actions.

The frontend was built with Next.js and TypeScript.

Originally, our frontend CI tasks ran sequentially, including linting, type checking, unit tests, and production build.

The entire job took about two minutes.

### Task

My goal was to reduce the CI execution time without removing any tests or quality checks.

### Action

First, I measured the existing pipeline using GitHub CLI and the GitHub Actions API.

I found that dependency installation and the Next.js build were the biggest bottlenecks.

Then, I split the pipeline into two parallel jobs: one for code checks and another for the production build.

After that, I added build caching using the GitHub Actions cache action.

This allowed the pipeline to reuse previous Next.js build data.

Finally, I ran the workflow again and checked the logs to confirm that the cache was successfully restored.

### Result

In my tests, the Next.js build time dropped from 66 seconds to 23 seconds, which was about a 65% reduction.

The Frontend Build job also dropped from 114 seconds to 59 seconds.

I learned that CI optimization should start with measuring the bottleneck, not just changing the configuration.

---

# 6. Common Interview Follow-up Questions

## Q1: Why did you split the pipeline into two jobs?

Because the code checks and production build were independent.

Running them in parallel reduced the time spent waiting for sequential tasks.

## Q2: What is the difference between a job and a step?

A job is a group of steps that run on the same runner.

Steps inside a job run sequentially, but independent jobs can run in parallel.

## Q3: What is the purpose of caching?

Caching allows us to reuse data from previous runs.

In my project, I used npm caching for downloaded dependencies and Next.js caching for intermediate build data.

## Q4: What is the difference between `key` and `restore-keys`?

The key identifies a specific cache.

If there is no exact match, restore-keys allows GitHub Actions to search for a compatible cache using a prefix.

## Q5: How did you know the cache was working?

I checked the GitHub Actions logs.

The first run showed a cache miss and successfully saved the cache.

The second run showed a cache hit and successfully restored it.

## Q6: Did caching guarantee a 65% improvement?

No. That was the improvement I observed in one test.

CI performance can vary depending on runner conditions and cache availability.

I would run more tests to measure the average improvement.

## Q7: What was the most challenging part?

One challenge was that the frontend jobs were skipped during testing.

I found that our path filter only detected changes inside the frontend directory.

Since I was modifying the CI workflow file, the frontend jobs were not triggered.

I fixed the filter and verified that both jobs ran successfully.

## Q8: Why did each parallel job install dependencies separately?

Because each GitHub Actions job runs in its own isolated environment.

The jobs do not automatically share installed dependencies.

Each job needs to check out the code and install the required packages.

## Q9: Why did you use `npm ci` instead of `npm install`?

Because `npm ci` is designed for automated environments.

It uses the lockfile to install dependencies consistently.

It also avoids modifying the lockfile during CI.

## Q10: What would you improve next?

I would run more CI tests to measure average performance.

I would also review the remaining bottlenecks and make sure the caching strategy stays effective as the project grows.

---

# 7. Technical Vocabulary

| Term | Simple English Explanation |
|---|---|
| CI Pipeline | An automated process that checks and builds code. |
| Workflow | An automated process defined in a YAML file. |
| Job | A group of steps executed on a runner. |
| Step | An individual task inside a job. |
| Runner | A machine that executes GitHub Actions jobs. |
| Sequential Execution | Tasks run one after another. |
| Parallel Jobs | Independent jobs run at the same time. |
| Bottleneck | The part of a process that limits overall performance. |
| Cache Hit | The system finds and reuses previously saved data. |
| Cache Miss | The system cannot find a matching cache. |
| Build Cache | Previously generated data that can speed up future builds. |
| Baseline | The original performance measurement used for comparison. |
| Path Filter | A rule that decides which changes should trigger a job. |
| Pull Request | A request to review and merge code changes. |

---

# 8. Three Sentences to Memorize

### Problem

> Our CI pipeline was slow because several tasks ran sequentially.

### Solution

> I split the pipeline into parallel jobs and added Next.js build caching.

### Result

> In my tests, the Next.js build time dropped from 66 seconds to 23 seconds, a reduction of about 65%.

---

# 9. Interview Practice Checklist

- [ ] Explain the original CI problem in 20 seconds.
- [ ] Explain why parallel jobs are faster.
- [ ] Explain the difference between jobs and steps.
- [ ] Explain why GitHub Actions needs caching.
- [ ] Explain the difference between npm cache and Next.js build cache.
- [ ] Explain `key` and `restore-keys`.
- [ ] Explain how to verify a cache hit.
- [ ] Explain the performance results without exaggerating.
- [ ] Describe the path filter issue and how it was fixed.
- [ ] Deliver the complete STAR story naturally in 90 seconds.

---

# 10. Final Takeaway

**Measure → Identify Bottleneck → Optimize → Verify → Document**

This is the main engineering process I learned from this project.

Instead of guessing what might improve performance, I measured the existing pipeline, made targeted changes, and verified the results using real CI data.