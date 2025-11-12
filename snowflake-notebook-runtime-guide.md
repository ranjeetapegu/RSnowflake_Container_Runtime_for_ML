# Snowflake Notebook Runtime Guide - CPU vs GPU

A simple guide to help you choose the right runtime for your Snowflake Notebooks.

---

## What are Runtime Types?

When you create a Snowflake Notebook on Container Runtime, you choose between two runtime environments:

### CPU Runtime
- **General-purpose computing** environment
- Standard processing power for everyday data science tasks
- Pre-installed with common Python packages (Pandas, NumPy, scikit-learn, XGBoost)
- **Lower cost** option

### GPU Runtime
- **Specialized computing** environment with Graphics Processing Units
- High-performance parallel processing for intensive workloads
- Includes all CPU packages **PLUS** deep learning frameworks (PyTorch, TensorFlow, CUDA)
- **Higher cost** but significantly faster for specific workloads

---

## When to Use CPU Runtime

Choose **CPU Runtime** for:

| Use Case | Why CPU? |
|----------|----------|
| **Data Exploration** | Analyzing datasets, creating visualizations, basic statistics |
| **Feature Engineering** | Data transformation, cleaning, and preparation |
| **Traditional ML Models** | Random Forest, XGBoost, LightGBM, Linear Regression |
| **Small to Medium Models** | Models with < 1 million parameters |
| **Development & Testing** | Prototyping before scaling to production |
| **Cost-Sensitive Projects** | When budget is a primary concern |

### CPU is Best When:
- Your model trains in **reasonable time** (minutes to a few hours)
- You're working with **tabular data**
- You're using **tree-based algorithms**
- You're doing **data analysis** more than model training

---

## When to Use GPU Runtime

Choose **GPU Runtime** for:

| Use Case | Why GPU? |
|----------|----------|
| **Deep Learning** | Neural networks with many layers |
| **Computer Vision** | Image classification, object detection, image generation |
| **Natural Language Processing** | Large language models (BERT, GPT, T5) |
| **LLM Fine-Tuning** | Customizing pre-trained language models |
| **Large-Scale Training** | Models with millions/billions of parameters |
| **Time-Critical Projects** | When faster training justifies higher cost |

### GPU is Best When:
- Training on CPU would take **hours/days** but GPU reduces it to **minutes/hours**
- You're working with **images, video, or text**
- You're using **PyTorch or TensorFlow** models
- You need **multiple training iterations** (hyperparameter tuning)

---

## Quick Decision Guide

```
Start Here: What type of data are you working with?

├─ Tabular/Structured Data (rows & columns)
│  └─ Use CPU Runtime
│
├─ Images, Video, or Audio
│  └─ Use GPU Runtime
│
└─ Text/Language Data
   ├─ Simple text analysis → CPU Runtime
   └─ LLMs or deep NLP models → GPU Runtime
```

---

## Cost Considerations

### Understanding the Trade-off

| Aspect | CPU | GPU |
|--------|-----|-----|
| **Hourly Rate** | Lower | Higher (3-10x more) |
| **Training Speed** | Baseline | 10-100x faster |
| **Best Value** | Long-running, low-intensity | Short-burst, high-intensity |

### Cost-Saving Tips

1. **Start with CPU** - Test your workflow first, switch to GPU only if needed
2. **Use GPU efficiently** - Run intensive training, then switch back to CPU for analysis
3. **Set auto-suspend** - Configure notebooks to shut down after idle time
4. **Right-size your pool** - Don't use large GPU instances for small models
5. **End sessions** - Always end your notebook session when done

---

## Best Practices

### 1. Choosing Your Runtime

**Rule of Thumb:**
- If you're unsure → **Start with CPU**
- If training is too slow → **Switch to GPU**
- If GPU doesn't speed up your work → **Stay with CPU**

### 2. Runtime Selection Strategy

```
Development Phase → CPU Runtime
   ↓ (test with small data/iterations)
Production Training → GPU Runtime (if needed)
   ↓ (full-scale training)
Model Deployment → CPU Runtime (for inference/monitoring)
```

### 3. Package Management

- **Check what's pre-installed** before installing new packages
- **You cannot change versions** of pre-installed packages
- **Install additional packages** only when needed
- **Request External Access Integration (EAI)** for PyPI or other repositories

### 4. Resource Management

**Essential Practices:**
- ✅ Set appropriate **idle timeout** (1-2 hours for active work)
- ✅ **End session** when done working
- ✅ Use **smaller compute pools** for development
- ✅ Monitor your **usage and costs** regularly
- ❌ Don't leave notebooks running overnight
- ❌ Don't use GPU for tasks that don't need it

### 5. Compute Pool Selection

| Notebook Type | Pool Size | When to Use |
|---------------|-----------|-------------|
| **Small CPU** | 2 vCPUs, 16GB RAM | Data exploration, testing |
| **Medium CPU** | 8 vCPUs, 64GB RAM | Production ML with traditional algorithms |
| **Small GPU** | 1 GPU, 24GB GPU RAM | Single-model training, fine-tuning |
| **Medium GPU** | 4 GPUs, 96GB GPU RAM | Distributed training, large models |

### 6. Session Management

**Before Starting:**
- Confirm you have the right runtime selected
- Verify external access is configured (if needed)
- Check that your compute pool is available

**During Work:**
- Monitor memory usage for large datasets
- Save your work frequently
- Document your notebook with clear comments

**After Completing:**
- End your session to free resources
- Review costs in your account usage dashboard
- Document runtime requirements for future use

---

## Common Scenarios

### Scenario 1: Data Analyst
**Need:** Explore sales data, create reports, build simple forecasts  
**Runtime:** CPU  
**Compute Pool:** Small or Medium CPU

### Scenario 2: ML Engineer (Traditional ML)
**Need:** Train XGBoost models for customer churn prediction  
**Runtime:** CPU  
**Compute Pool:** Medium CPU

### Scenario 3: ML Engineer (Deep Learning)
**Need:** Train a computer vision model for product classification  
**Runtime:** GPU  
**Compute Pool:** Small or Medium GPU

### Scenario 4: Data Scientist (LLM Work)
**Need:** Fine-tune a BERT model for sentiment analysis  
**Runtime:** GPU  
**Compute Pool:** Small GPU (or Medium for larger models)

### Scenario 5: Research Team
**Need:** Experiment with different model architectures  
**Runtime:** Start CPU → Switch to GPU for final training  
**Compute Pool:** Small CPU for experiments, Medium GPU for training

---

## Key Takeaways

1. **CPU is your default choice** - Use for most data science tasks
2. **GPU is for specialized workloads** - Deep learning, computer vision, NLP
3. **Cost scales with power** - GPU is more expensive, use it wisely
4. **You can switch runtimes** - Start small, scale up as needed
5. **Always end sessions** - Don't waste resources when not working
6. **Monitor your usage** - Keep track of costs and optimize accordingly

---

## Need Help Deciding?

Ask yourself these questions:

1. **What type of model am I building?**
   - Traditional ML (Decision Trees, Regression) → CPU
   - Deep Neural Networks → GPU

2. **How long does training take on CPU?**
   - Minutes to 1-2 hours → CPU is fine
   - Hours to days → Consider GPU

3. **What's my budget?**
   - Limited budget → Start with CPU
   - Flexible budget + time-sensitive → GPU

4. **How many iterations do I need?**
   - Few iterations → CPU might be sufficient
   - Many iterations (hyperparameter tuning) → GPU saves time

5. **What frameworks am I using?**
   - scikit-learn, XGBoost, Pandas → CPU
   - PyTorch, TensorFlow for deep learning → GPU

---

## Additional Resources

- **Snowflake Documentation**: [Notebooks on Container Runtime](https://docs.snowflake.com/en/developer-guide/snowflake-ml/notebooks-on-spcs)
- **Container Runtime for ML**: [Overview](https://docs.snowflake.com/en/developer-guide/snowflake-ml/container-runtime-ml)
- **External Access Setup**: See `snowflake-external-access-setup.md` in this repository

---

## Quick Reference Card

| Question | CPU | GPU |
|----------|-----|-----|
| Data type? | Tabular/Structured | Images/Video/Text |
| Model type? | Traditional ML | Deep Learning |
| Framework? | scikit-learn/XGBoost | PyTorch/TensorFlow |
| Training time? | Minutes-Hours | Hours-Days on CPU |
| Budget? | Cost-conscious | Time-sensitive |
| Use case? | Analysis/Feature Eng | Neural Networks |

---

*Last Updated: November 12, 2025*  
*Reference: [Snowflake Notebooks on Container Runtime Documentation](https://docs.snowflake.com/en/developer-guide/snowflake-ml/notebooks-on-spcs)*

