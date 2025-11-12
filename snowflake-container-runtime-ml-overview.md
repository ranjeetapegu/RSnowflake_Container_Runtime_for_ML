## Snowflake Container Runtime for ML — Customer Overview

### Who is this for
- **Data and ML teams** modernizing model development and training on governed data
- **Platform/IT** standardizing secure, scalable ML environments inside Snowflake

### What it is
- **Preconfigured, customizable container environments** for ML running on Snowpark Container Services
- Built for **interactive notebooks and batch ML workloads** (training, tuning, batch inference, fine-tuning)
- Supports **CPU and GPU compute pools**, integrated with **Snowflake Notebooks**

Reference: Snowflake docs — Container Runtime for ML: `https://docs.snowflake.com/en/developer-guide/snowflake-ml/container-runtime-ml`

### Why choose it
- **End-to-end inside Snowflake**: Develop, train, and deploy without moving data, preserving governance
- **Performance at scale**: Distributed processing that uses all GPUs on multi-GPU nodes by default
- **Security and compliance**: Data stays within Snowflake’s managed boundary and policies
- **Fast onboarding**: Popular ML/DL frameworks preinstalled; install extras with `pip` as needed
- **Operational simplicity**: Standardized environments, jobs/pipelines, observability, model registry and serving
- **Cost control**: Choose compute pools and scale resources per task, then shut down when done

### Key capabilities
- **Execution environment**: Rich, curated ML stacks for CPU/GPU; extend via `pip`
- **Distributed processing**: Snowflake ML’s framework maximizes utilization across available GPUs/CPUs
- **Optimized data loading**: `snowflake.ml.data` connectors stream from Snowflake tables/DataFrames/Datasets into TensorFlow, PyTorch, or Pandas
- **Build with open source**: Use familiar libraries (PyTorch, TensorFlow, scikit-learn, XGBoost, etc.) inside Snowflake
- **Optimized training APIs**: Distributed XGBoost, LightGBM, and PyTorch with scaling configs
- **Model lifecycle**: Registry, serving (warehouse or container services), pipelines, jobs, observability

### How it works (at a glance)
- Workloads execute in **Snowpark Container Services** on CPU/GPU compute pools
- The ML runtime **auto-distributes** data loading and training; you can **configure scaling** (GPUs, memory) per task
- **Data connectors** convert Snowflake DataFrames/Datasets to TensorFlow/PyTorch/Pandas for training
- Use with **Snowflake Notebooks** for an end-to-end dev experience

### When to use
- **Large-scale training** or inference that benefits from multi-GPU and distributed processing
- **Governed ML** where data must remain in Snowflake for security/compliance
- **Standardizing environments** across teams to reduce “works on my machine” and devops toil

### Getting started (typical steps)
1. Enable Snowpark Container Services and create a **compute pool** (CPU/GPU)
2. Launch a **Notebook on Container Runtime for ML**
3. Access data via **Snowpark**; connect to frameworks using **DataConnector**
4. **Train** with open-source libraries or Snowflake’s **distributed training APIs**
5. **Register and serve** models or schedule **ML Jobs** / pipelines

### Example: Load data and train with XGBoost
```python
from snowflake.snowpark.context import get_active_session
from snowflake.ml.data.data_connector import DataConnector
import xgboost as xgb

session = get_active_session()
df = session.table("MY_DATASET")
pdf = DataConnector.from_dataframe(df).to_pandas()

X = pdf[["feature1", "feature2"]]
y = pdf["label"]
model = xgb.XGBClassifier().fit(X, y)
```

### Considerations and FAQs
- **Packages**: CPU and GPU images differ; popular frameworks are preinstalled. You can `pip install` additional packages if allowed by policy
- **Scaling**: Default uses all GPUs on multi-GPU nodes; tune via scaling configs per task
- **Costs**: Billed by compute resources; stop pools/notebooks when idle
- **Migrations**: Minimal app code changes to adopt distributed APIs; keep your preferred frameworks

### Related features to highlight
- **Distributed training**: XGBoost, LightGBM, PyTorch distributors and scaling configs
- **Model Serving**: In-warehouse or via Snowpark Container Services
- **Observability**: Metrics, logs, lineage, and explainability tooling
- **Integrations**: Ray scaling and CUDA-X libraries (GPU acceleration)

### Links
- Official docs — Container Runtime for ML: `https://docs.snowflake.com/en/developer-guide/snowflake-ml/container-runtime-ml`


