# Docker Build Optimizations & Caching Strategies

The goal of this repo is to demonstrate various Docker build optimization techniques using multi-stage builds, Next.js Output Standalone, and caching strategies.

We use a sample Next.js application with a set of "dummy" packages added to `package.json` (e.g., `@reduxjs/toolkit`, `framer-motion`, `axios`, `zod`, `lucide-react`) to simulate a real-world application with heavy dependencies. This helps to better visualize the impact of caching and optimization strategies on build times and image sizes.

## Scenarios & Results

We explored three different build scenarios to demonstrate the impact of caching and build strategies.

### 1. Build Default (Baseline)

This is a standard build that includes all dependencies.

* **Workflow**: `.github/workflows/build-default.yml`
* **Dockerfile**: `build/default/Dockerfile`

**Metrics:**

| Metric            | Value      |
| ----------------- | ---------- |
| Docker Build Time | 30 seconds |
| Docker Image Size | 1.1GB      |
| Total GHA Runtime | 40s        |

_Note: In this scenario, the npm cache was also stored in the image, contributing to the larger size._

---

### 2. Build (npm Cache)

This approach leverages Docker BuildKit's cache mounts to cache npm packages locally during the build.

* **Workflow**: `.github/workflows/build-npm-cache.yml`
* **Dockerfile**: `build/cached/Dockerfile.npmcache`

**Metrics:**

**Initial Run:**

| Metric            | Value       |
| ----------------- | ----------- |
| Docker Build Time | 65 seconds  |
| Docker Image Size | 918MB       |
| Total GHA Runtime | 1m 20s      |

**Subsequent Runs:**

| Metric            | Value      |
| ----------------- | ---------- |
| Docker Build Time | 30 seconds |
| Docker Image Size | 918MB      |
| Total GHA Runtime | 50s        |

---

### 3. Build Standalone (npm Cache)

This method combines npm caching with Next.js `output: 'standalone'` feature. This significantly reduces the final image size by only including the necessary files for production execution, automatically tracing dependencies.

* **Workflow**: `.github/workflows/build-cache-standalone.yml`
* **Dockerfile**: `build/cached/Dockerfile.standalone`

**Metrics:**

**Initial Run:**

| Metric            | Value       |
| ----------------- | ----------- |
| Docker Build Time | 54 seconds  |
| Docker Image Size | 300MB       |
| Total GHA Runtime | 1m 15s      |

**Subsequent Runs:**

| Metric            | Value      |
| ----------------- | ---------- |
| Docker Build Time | 18 seconds |
| Docker Image Size | 300MB      |
| Total GHA Runtime | 40s        |

_Observation: Massive image size reduction from 1.1GB to 300MB using standalone mode, with the fastest subsequent build times._

## Conclusions

1.  **Next.js Standalone Output** is critical for reducing image size (reduced from 1.1GB to 300MB), making deployments faster and storage cheaper.
2.  **npm Caching** significantly improves build times on subsequent runs by avoiding re-downloading packages.
3.  Combining **Standalone Output** with **npm Caching** yields the best results in terms of both image size and build performance.

## Disclaimer

These results are examples based on a specific sample application and environment. Actual timing and sizes will vary depending on your dependencies, network speed, and GitHub Actions runner performance.

