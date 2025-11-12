# How to Create a GPU Compute Pool in Snowflake

A simple step-by-step guide for creating GPU compute pools for Snowflake Notebooks.

---

## What is a GPU Compute Pool?

A GPU compute pool is a collection of virtual machines with Graphics Processing Units (GPUs) that provide the computing power for running your Snowflake Notebooks on Container Runtime.

**Key Points:**
- Required for running notebooks on GPU runtime
- Uses Snowpark Container Services (SPCS)
- Each pool can have multiple nodes
- Each node can run one notebook per user at a time

---

## Prerequisites

### 1. Required Role and Privileges

You need one of these roles:
- `ACCOUNTADMIN` (recommended for setup)
- A custom role with `CREATE COMPUTE POOL` privilege

**Check your current role:**
```sql
SELECT CURRENT_ROLE();
```

**Switch to admin role if needed:**
```sql
USE ROLE ACCOUNTADMIN;
```

### 2. Enable Snowpark Container Services

Snowpark Container Services must be enabled in your account (usually enabled by default in newer accounts).

---

## GPU Pool Types

Snowflake offers different GPU instance families:

| Instance Family | GPUs | GPU Memory | vCPUs | RAM | Best For |
|----------------|------|------------|-------|-----|----------|
| **GPU_NV_S** | 1 | 24 GB | 4 | 32 GB | Single-GPU training, fine-tuning |
| **GPU_NV_M** | 4 | 96 GB (total) | 16 | 128 GB | Multi-GPU training, larger models |
| **GPU_NV_L** | 8 | 192 GB (total) | 32 | 256 GB | Large-scale training, LLMs |

**Recommendation:** Start with `GPU_NV_S` for most ML workloads.

---

## Step-by-Step: Create a GPU Compute Pool

### Step 1: Connect to Snowflake

Open Snowsight (Snowflake's web interface) or use your SQL client.

### Step 2: Set Your Context

```sql
-- Use admin role
USE ROLE ACCOUNTADMIN;

-- Optionally create or use a database for organization
USE DATABASE MY_DATABASE;
USE SCHEMA MY_SCHEMA;
```

### Step 3: Create Your First GPU Pool

**Basic GPU Pool (Small - Recommended to Start):**

```sql
CREATE COMPUTE POOL MY_GPU_POOL
  MIN_NODES = 0
  MAX_NODES = 3
  INSTANCE_FAMILY = GPU_NV_S
  AUTO_RESUME = TRUE
  AUTO_SUSPEND_SECS = 600;
```

**What each parameter means:**
- `MIN_NODES = 0`: No nodes running when idle (saves cost)
- `MAX_NODES = 3`: Up to 3 users can run notebooks simultaneously
- `INSTANCE_FAMILY = GPU_NV_S`: Small GPU (1 GPU per node)
- `AUTO_RESUME = TRUE`: Starts automatically when needed
- `AUTO_SUSPEND_SECS = 600`: Shuts down after 10 minutes of inactivity

### Step 4: Grant Access to Users

```sql
-- Grant usage to your data science team role
GRANT USAGE ON COMPUTE POOL MY_GPU_POOL TO ROLE DATA_SCIENTIST;

-- Or grant to specific user roles
GRANT USAGE ON COMPUTE POOL MY_GPU_POOL TO ROLE ML_ENGINEER;
```

### Step 5: Verify the Pool

```sql
-- View all compute pools
SHOW COMPUTE POOLS;

-- Check detailed information
DESC COMPUTE POOL MY_GPU_POOL;

-- Check pool status
SELECT * FROM TABLE(
  INFORMATION_SCHEMA.COMPUTE_POOL_STATUS('MY_GPU_POOL')
);
```

---

## Common GPU Pool Configurations

### Configuration 1: Development/Testing Pool

**For:** Individual developers testing ML models

```sql
CREATE COMPUTE POOL GPU_DEV_POOL
  MIN_NODES = 0
  MAX_NODES = 2
  INSTANCE_FAMILY = GPU_NV_S
  AUTO_RESUME = TRUE
  AUTO_SUSPEND_SECS = 300;  -- 5 minutes

GRANT USAGE ON COMPUTE POOL GPU_DEV_POOL TO ROLE DATA_SCIENTIST;
```

### Configuration 2: Production Training Pool

**For:** Team of data scientists running production models

```sql
CREATE COMPUTE POOL GPU_PROD_POOL
  MIN_NODES = 0
  MAX_NODES = 5
  INSTANCE_FAMILY = GPU_NV_M
  AUTO_RESUME = TRUE
  AUTO_SUSPEND_SECS = 600;  -- 10 minutes

GRANT USAGE ON COMPUTE POOL GPU_PROD_POOL TO ROLE ML_ENGINEER;
```

### Configuration 3: Large-Scale Training Pool

**For:** LLM training or very large models

```sql
CREATE COMPUTE POOL GPU_LARGE_POOL
  MIN_NODES = 0
  MAX_NODES = 3
  INSTANCE_FAMILY = GPU_NV_L
  AUTO_RESUME = TRUE
  AUTO_SUSPEND_SECS = 900;  -- 15 minutes

GRANT USAGE ON COMPUTE POOL GPU_LARGE_POOL TO ROLE ML_ENGINEER;
```

---

## Using Your GPU Pool in a Notebook

### Option 1: When Creating a New Notebook

1. In Snowsight, go to **Projects** → **Notebooks**
2. Click **+ Notebook**
3. Fill in the details:
   - **Name:** My ML Notebook
   - **Database:** Choose your database
   - **Schema:** Choose your schema
   - **Runtime:** Select **Run on container**
   - **Runtime Version:** Select **GPU**
   - **Compute Pool:** Select **MY_GPU_POOL**
   - **Warehouse:** Select a warehouse for SQL queries
4. Click **Create**

### Option 2: Update an Existing Notebook

```sql
ALTER NOTEBOOK MY_NOTEBOOK
  SET COMPUTE_POOL = MY_GPU_POOL;
```

Or in Snowsight:
1. Open your notebook
2. Click the **⋮** (three dots) menu
3. Select **Notebook settings**
4. Under **Compute pool**, select your GPU pool
5. Click **Save**

---

## Managing Your GPU Pool

### Check Pool Status

```sql
-- View all pools
SHOW COMPUTE POOLS;

-- Check if pool is running
SELECT * FROM TABLE(
  INFORMATION_SCHEMA.COMPUTE_POOL_STATUS('MY_GPU_POOL')
);
```

### Suspend Pool Manually

```sql
ALTER COMPUTE POOL MY_GPU_POOL SUSPEND;
```

### Resume Pool Manually

```sql
ALTER COMPUTE POOL MY_GPU_POOL RESUME;
```

### Modify Pool Settings

```sql
-- Increase max nodes
ALTER COMPUTE POOL MY_GPU_POOL SET MAX_NODES = 5;

-- Change auto-suspend time (in seconds)
ALTER COMPUTE POOL MY_GPU_POOL SET AUTO_SUSPEND_SECS = 900;
```

### Drop a Pool (if needed)

```sql
-- Pool must be empty and suspended first
ALTER COMPUTE POOL MY_GPU_POOL SUSPEND;

-- Then drop it
DROP COMPUTE POOL MY_GPU_POOL;
```

---

## Important Considerations

### 1. Node Limits
- **Each node runs ONE notebook per user**
- Set `MAX_NODES` based on your team size
- Example: Team of 5 users = set `MAX_NODES` to at least 5

### 2. Cost Management
- GPU pools are **more expensive** than CPU pools
- Always set `MIN_NODES = 0` to avoid idle costs
- Use `AUTO_SUSPEND_SECS` to shut down quickly when idle
- **Monitor your usage regularly**

### 3. Auto-Suspend Times

| Use Case | Recommended Time |
|----------|-----------------|
| Development | 300-600 seconds (5-10 min) |
| Production Training | 600-900 seconds (10-15 min) |
| Scheduled Jobs | 60-300 seconds (1-5 min) |

### 4. Choosing the Right Size

| Your Need | Recommended Pool |
|-----------|-----------------|
| Learning & testing | GPU_NV_S (1 GPU) |
| Single model training | GPU_NV_S (1 GPU) |
| Multiple model versions | GPU_NV_M (4 GPUs) |
| Large models or LLMs | GPU_NV_M or GPU_NV_L |
| Distributed training | GPU_NV_M or GPU_NV_L |

---

## Troubleshooting

### Issue 1: "Insufficient privileges" Error

**Solution:**
```sql
-- Make sure you're using the right role
USE ROLE ACCOUNTADMIN;

-- Or grant privilege to your role
GRANT CREATE COMPUTE POOL ON ACCOUNT TO ROLE MY_ROLE;
```

### Issue 2: Pool Won't Start

**Solution:**
```sql
-- Check pool status
SHOW COMPUTE POOLS;

-- Try resuming manually
ALTER COMPUTE POOL MY_GPU_POOL RESUME;

-- Check for errors in activity history
```

### Issue 3: "No available nodes" Error

**Solution:**
```sql
-- Increase MAX_NODES
ALTER COMPUTE POOL MY_GPU_POOL SET MAX_NODES = 5;

-- Or create an additional pool
```

### Issue 4: High Costs

**Solution:**
```sql
-- Check current settings
DESC COMPUTE POOL MY_GPU_POOL;

-- Set aggressive auto-suspend
ALTER COMPUTE POOL MY_GPU_POOL SET AUTO_SUSPEND_SECS = 300;

-- Reduce max nodes if not all are needed
ALTER COMPUTE POOL MY_GPU_POOL SET MAX_NODES = 2;

-- Ensure MIN_NODES = 0
ALTER COMPUTE POOL MY_GPU_POOL SET MIN_NODES = 0;
```

---

## Best Practices Checklist

Before creating your GPU pool, review this checklist:

- [ ] Start with `GPU_NV_S` unless you know you need larger
- [ ] Set `MIN_NODES = 0` to avoid idle costs
- [ ] Set `MAX_NODES` to at least your team size
- [ ] Configure `AUTO_SUSPEND_SECS` to 10 minutes or less
- [ ] Grant access only to roles that need it
- [ ] Document pool purpose and expected usage
- [ ] Set up cost monitoring and alerts
- [ ] Train users to end notebook sessions when done

---

## Monitoring Costs

### Check GPU Pool Usage

```sql
-- View compute usage history
SELECT 
  START_TIME,
  END_TIME,
  COMPUTE_POOL_NAME,
  CREDITS_USED,
  DATEDIFF('minute', START_TIME, END_TIME) as MINUTES_USED
FROM SNOWFLAKE.ACCOUNT_USAGE.METERING_HISTORY
WHERE SERVICE_TYPE = 'SNOWPARK_CONTAINER_SERVICES'
  AND COMPUTE_POOL_NAME = 'MY_GPU_POOL'
  AND START_TIME >= DATEADD(day, -7, CURRENT_TIMESTAMP())
ORDER BY START_TIME DESC;
```

### Set Up Cost Alerts

Work with your account admin to set up resource monitors:

```sql
-- Create a resource monitor (ACCOUNTADMIN only)
CREATE RESOURCE MONITOR GPU_COST_MONITOR
  WITH CREDIT_QUOTA = 100
  FREQUENCY = MONTHLY
  START_TIMESTAMP = IMMEDIATELY
  TRIGGERS 
    ON 75 PERCENT DO NOTIFY
    ON 90 PERCENT DO SUSPEND
    ON 100 PERCENT DO SUSPEND_IMMEDIATE;
```

---

## Quick Start Template

Copy and customize this template for your needs:

```sql
-- 1. Set context
USE ROLE ACCOUNTADMIN;
USE DATABASE MY_DATABASE;
USE SCHEMA MY_SCHEMA;

-- 2. Create GPU pool
CREATE COMPUTE POOL MY_GPU_POOL
  MIN_NODES = 0
  MAX_NODES = 3                    -- Adjust for your team size
  INSTANCE_FAMILY = GPU_NV_S       -- Start small
  AUTO_RESUME = TRUE
  AUTO_SUSPEND_SECS = 600;         -- 10 minutes

-- 3. Grant access
GRANT USAGE ON COMPUTE POOL MY_GPU_POOL TO ROLE DATA_SCIENTIST;

-- 4. Verify
SHOW COMPUTE POOLS;
DESC COMPUTE POOL MY_GPU_POOL;

-- 5. Test in notebook
-- Go to Snowsight → Projects → Notebooks
-- Create notebook with GPU runtime and select MY_GPU_POOL
```

---

## Next Steps

After creating your GPU pool:

1. **Create a test notebook** using the GPU runtime
2. **Verify GPU is available** in the notebook:
   ```python
   import torch
   print(f"GPU Available: {torch.cuda.is_available()}")
   print(f"GPU Name: {torch.cuda.get_device_name(0)}")
   ```
3. **Train a small model** to test the setup
4. **Monitor costs** in the first week
5. **Adjust settings** based on usage patterns

---

## Summary

**To create a GPU pool, you need:**
1. ACCOUNTADMIN role (or equivalent privileges)
2. A SQL command with pool configuration
3. Grant USAGE to your team roles
4. Use the pool in your notebooks

**Remember:**
- Start small with `GPU_NV_S`
- Set `MIN_NODES = 0` to control costs
- Use `AUTO_SUSPEND_SECS` to shut down quickly
- Always end notebook sessions when done
- Monitor your usage and costs regularly

---

## Additional Resources

- **Runtime Selection Guide:** See `snowflake-notebook-runtime-guide.md`
- **External Access Setup:** See `snowflake-external-access-setup.md`
- **Snowflake Documentation:** [Compute Pools](https://docs.snowflake.com/en/developer-guide/snowpark-container-services/working-with-compute-pool)

---

*Last Updated: November 12, 2025*  
*Reference: [Snowflake Notebooks on Container Runtime](https://docs.snowflake.com/en/developer-guide/snowflake-ml/notebooks-on-spcs)*

