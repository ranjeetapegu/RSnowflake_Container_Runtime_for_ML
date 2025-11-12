# Snowflake Notebooks External Access Setup Guide

Complete guide for configuring External Access Integrations (EAI) for Snowflake Notebooks, with focus on PyPI integration.

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Understanding External Access Components](#understanding-external-access-components)
4. [Step-by-Step Setup for PyPI](#step-by-step-setup-for-pypi)
5. [Enabling EAI in Snowsight](#enabling-eai-in-snowsight)
6. [Using External Access in Notebooks](#using-external-access-in-notebooks)
7. [Additional EAI Examples](#additional-eai-examples)
8. [Working with Secrets](#working-with-secrets)
9. [Troubleshooting](#troubleshooting)
10. [Best Practices](#best-practices)

---

## Overview

### What is External Access Integration (EAI)?

By default, Snowflake restricts network traffic from external endpoints for security. When working with Snowflake Notebooks, you may need to:
- Install packages from PyPI
- Call external APIs (OpenAI, Hugging Face, etc.)
- Access external data sources
- Authenticate with third-party services

External Access Integrations (EAI) allow you to securely access external endpoints from within Snowflake Notebooks.

### Why Do You Need EAI?

- **Security**: Keep sensitive credentials secure using Snowflake secrets instead of hardcoding
- **Network Access**: Enable controlled access to external services
- **Authentication**: Manage API keys, tokens, and credentials securely
- **Compliance**: Maintain audit trails of external access

---

## Prerequisites

### Required Privileges

To create and manage EAIs, you need:

- `CREATE INTEGRATION` privilege on the account
- `CREATE SECRET` privilege (if using secrets)
- `CREATE NETWORK RULE` privilege
- Organization administrator role or appropriate grants

### Verify Your Role

```sql
-- Check current role
SELECT CURRENT_ROLE();

-- Check available roles
SHOW ROLES;

-- Switch to appropriate role
USE ROLE ACCOUNTADMIN;  -- or your admin role
```

---

## Understanding External Access Components

External Access consists of three main components:

### 1. Network Rules

Define which external endpoints are allowed.

```sql
CREATE OR REPLACE NETWORK RULE <rule_name>
  MODE = EGRESS                    -- Outbound traffic
  TYPE = HOST_PORT                 -- Host and port based
  VALUE_LIST = ('host1.com', 'host2.com');  -- Allowed hosts
```

### 2. External Access Integration

Links network rules with secrets and enables access.

```sql
CREATE OR REPLACE EXTERNAL ACCESS INTEGRATION <integration_name>
  ALLOWED_NETWORK_RULES = (<rule_name>)
  ALLOWED_AUTHENTICATION_SECRETS = (<secret_name>)  -- Optional
  ENABLED = true;
```

### 3. Secrets (Optional)

Store sensitive credentials like API keys, passwords, and tokens.

```sql
CREATE SECRET <secret_name>
  TYPE = GENERIC_STRING
  SECRET_STRING = '<your-api-key>';
```

---

## Step-by-Step Setup for PyPI

### Why PyPI EAI is Needed

PyPI (Python Package Index) is required when you need to install Python packages in your Snowflake Notebooks that are not pre-installed in the Snowflake environment.

### Step 1: Switch to Admin Role

```sql
-- Use a role with CREATE INTEGRATION privileges
USE ROLE ACCOUNTADMIN;

-- Or create a custom role with required privileges
USE ROLE SYSADMIN;
```

### Step 2: Create Network Rule for PyPI

PyPI requires access to multiple domains for package downloads:

```sql
-- Create network rule for PyPI access
CREATE OR REPLACE NETWORK RULE pypi_network_rule
  MODE = EGRESS
  TYPE = HOST_PORT
  VALUE_LIST = (
    'pypi.org',                    -- Main PyPI site
    'pypi.python.org',             -- Legacy PyPI domain
    'pythonhosted.org',            -- CDN for packages
    'files.pythonhosted.org'       -- File downloads
  );
```

**Important:** All four domains are required for PyPI to work properly:
- `pypi.org`: Main package index
- `pypi.python.org`: Legacy support
- `pythonhosted.org`: Content delivery
- `files.pythonhosted.org`: Actual package files

### Step 3: Create External Access Integration

```sql
-- Create the external access integration
CREATE OR REPLACE EXTERNAL ACCESS INTEGRATION pypi_access_integration
  ALLOWED_NETWORK_RULES = (pypi_network_rule)
  ENABLED = true;
```

**Note:** PyPI typically doesn't require authentication secrets, so we only specify the network rule.

### Step 4: Verify the Integration

```sql
-- Show the created integration
SHOW EXTERNAL ACCESS INTEGRATIONS;

-- Describe the integration details
DESC INTEGRATION pypi_access_integration;
```

### Step 5: Grant USAGE Privileges

Grant the USAGE privilege to roles that will use the integration in notebooks:

```sql
-- Grant to a specific role
GRANT USAGE ON INTEGRATION pypi_access_integration TO ROLE DATA_SCIENTIST;

-- Or grant to multiple roles
GRANT USAGE ON INTEGRATION pypi_access_integration TO ROLE ML_ENGINEER;
GRANT USAGE ON INTEGRATION pypi_access_integration TO ROLE ANALYST;
```

**Important:** 
- The role used to create/run the notebook MUST have USAGE privilege
- Granting to PUBLIC role will NOT work
- Each user role needs explicit USAGE grant

### Step 6: Associate EAI with Notebook

```sql
-- Associate the integration with your notebook
ALTER NOTEBOOK my_ml_notebook
  SET EXTERNAL_ACCESS_INTEGRATIONS = (pypi_access_integration);
```

---

## Enabling EAI in Snowsight

After creating the EAI via SQL, you need to enable it in the Snowsight UI:

### Step-by-Step in Snowsight

1. **Navigate to Notebooks**
   - Click on **Projects** in the left navigation menu
   - Select **Notebooks**

2. **Open Your Notebook**
   - Click on the notebook where you want to enable external access

3. **Restart the Notebook Session** (Important!)
   - You must restart the session to see newly created EAIs
   - Click the restart button or run: `st.experimental_rerun()`

4. **Access Notebook Settings**
   - Click the **three dots** (⋮) menu at the top right
   - Select **Notebook settings**

5. **Enable External Access**
   - Click on the **External access** tab
   - Toggle **ON** the `pypi_access_integration` (or your EAI name)
   - Click **Save**

### Verification in Notebook

After enabling, verify in a notebook cell:

```python
# Test PyPI access by installing a package
import sys
import subprocess

# Try installing a package
subprocess.check_call([sys.executable, "-m", "pip", "install", "requests"])
print("✓ PyPI access working!")
```

---

## Using External Access in Notebooks

### Installing Packages from PyPI

Once PyPI EAI is enabled, you can install packages:

```python
# Method 1: Using pip in a cell
import sys
import subprocess

subprocess.check_call([sys.executable, "-m", "pip", "install", "pandas==2.0.0"])
subprocess.check_call([sys.executable, "-m", "pip", "install", "scikit-learn"])
subprocess.check_call([sys.executable, "-m", "pip", "install", "matplotlib"])
```

```python
# Method 2: Install multiple packages
packages = ['numpy', 'scipy', 'seaborn']
for package in packages:
    subprocess.check_call([sys.executable, "-m", "pip", "install", package])
```

```python
# Method 3: Install from requirements
requirements = """
pandas>=2.0.0
numpy>=1.24.0
scikit-learn>=1.3.0
matplotlib>=3.7.0
"""

with open('requirements.txt', 'w') as f:
    f.write(requirements)

subprocess.check_call([sys.executable, "-m", "pip", "install", "-r", "requirements.txt"])
```

### Best Practices for Package Installation

```python
def install_packages(packages):
    """
    Install packages with error handling
    """
    import sys
    import subprocess
    
    for package in packages:
        try:
            subprocess.check_call([
                sys.executable, "-m", "pip", "install", 
                package, "--quiet"
            ])
            print(f"✓ {package} installed successfully")
        except subprocess.CalledProcessError as e:
            print(f"✗ Failed to install {package}: {e}")

# Usage
install_packages(['requests', 'beautifulsoup4', 'lxml'])
```

---

## Additional EAI Examples

### 1. OpenAI API Access

Complete setup with secrets for API authentication:

```sql
-- Step 1: Create a secret for the API key
CREATE SECRET openai_key
  TYPE = GENERIC_STRING
  SECRET_STRING = 'sk-your-api-key-here';

-- Step 2: Create network rule
CREATE OR REPLACE NETWORK RULE openai_rule
  MODE = EGRESS
  TYPE = HOST_PORT
  VALUE_LIST = ('api.openai.com');

-- Step 3: Create external access integration
CREATE OR REPLACE EXTERNAL ACCESS INTEGRATION openai_integration
  ALLOWED_NETWORK_RULES = (openai_rule)
  ALLOWED_AUTHENTICATION_SECRETS = (openai_key)
  ENABLED = true;

-- Step 4: Grant privileges
GRANT USAGE ON INTEGRATION openai_integration TO ROLE DATA_SCIENTIST;

-- Step 5: Associate with notebook
ALTER NOTEBOOK my_ai_notebook
  SET EXTERNAL_ACCESS_INTEGRATIONS = (openai_integration),
    SECRETS = ('openai_key' = openai_key);
```

**Using in Notebook:**

```python
import streamlit as st
import openai

# Access the secret
api_key = st.secrets['openai_key']
openai.api_key = api_key

# Make API calls
response = openai.ChatCompletion.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "Hello!"}]
)
print(response.choices[0].message.content)
```

### 2. Hugging Face Access

```sql
-- Create network rule for Hugging Face
CREATE OR REPLACE NETWORK RULE hf_network_rule
  MODE = EGRESS
  TYPE = HOST_PORT
  VALUE_LIST = (
    'huggingface.co',
    'cdn-lfs.huggingface.co'
  );

-- Create external access integration
CREATE OR REPLACE EXTERNAL ACCESS INTEGRATION hf_access_integration
  ALLOWED_NETWORK_RULES = (hf_network_rule)
  ENABLED = true;

-- Grant privileges
GRANT USAGE ON INTEGRATION hf_access_integration TO ROLE ML_ENGINEER;

-- Associate with notebook
ALTER NOTEBOOK my_hf_notebook
  SET EXTERNAL_ACCESS_INTEGRATIONS = (hf_access_integration);
```

**Using in Notebook:**

```python
from transformers import pipeline

# Download and use Hugging Face models
classifier = pipeline("sentiment-analysis")
result = classifier("I love Snowflake Notebooks!")
print(result)
```

### 3. GitHub API Access

```sql
-- Create secret for GitHub token
CREATE SECRET github_token
  TYPE = PASSWORD
  USERNAME = 'your_github_username'
  PASSWORD = 'ghp_your_github_token';

-- Create network rule
CREATE OR REPLACE NETWORK RULE github_rule
  MODE = EGRESS
  TYPE = HOST_PORT
  VALUE_LIST = ('api.github.com', 'github.com');

-- Create external access integration
CREATE OR REPLACE EXTERNAL ACCESS INTEGRATION github_integration
  ALLOWED_NETWORK_RULES = (github_rule)
  ALLOWED_AUTHENTICATION_SECRETS = (github_token)
  ENABLED = true;

-- Grant privileges
GRANT USAGE ON INTEGRATION github_integration TO ROLE DEVELOPER;

-- Associate with notebook
ALTER NOTEBOOK my_github_notebook
  SET EXTERNAL_ACCESS_INTEGRATIONS = (github_integration),
    SECRETS = ('cred' = github_token);
```

**Using in Notebook:**

```python
import streamlit as st
import requests
from requests.auth import HTTPBasicAuth

# Access credentials
username = st.secrets.cred.username
password = st.secrets.cred.password

# Make authenticated request
response = requests.get(
    'https://api.github.com/user',
    auth=HTTPBasicAuth(username, password)
)

print(f"Status: {response.status_code}")
print(f"User: {response.json()['login']}")
```

### 4. Combined EAI Setup

Multiple integrations for a single notebook:

```sql
-- Associate multiple integrations
ALTER NOTEBOOK my_comprehensive_notebook
  SET EXTERNAL_ACCESS_INTEGRATIONS = (
    pypi_access_integration,
    openai_integration,
    hf_access_integration
  ),
  SECRETS = (
    'openai_key' = openai_key
  );
```

---

## Working with Secrets

### Secret Types

#### 1. GENERIC_STRING

For single-value secrets like API keys:

```sql
CREATE SECRET my_api_key
  TYPE = GENERIC_STRING
  SECRET_STRING = 'your-api-key-value';
```

**Access in Notebook:**

```python
import streamlit as st

# Method 1: Dictionary access
api_key = st.secrets['my_api_key']

# Method 2: Attribute access
api_key = st.secrets.my_api_key
```

#### 2. PASSWORD

For username/password pairs:

```sql
CREATE SECRET my_credentials
  TYPE = PASSWORD
  USERNAME = 'my_username'
  PASSWORD = 'my_password';
```

**Access in Notebook:**

```python
import streamlit as st

username = st.secrets.my_credentials.username
password = st.secrets.my_credentials.password
```

#### 3. OAUTH2

For OAuth tokens:

```sql
CREATE OR REPLACE SECRET oauth_token
  TYPE = OAUTH2
  API_AUTHENTICATION = my_oauth_integration
  OAUTH_REFRESH_TOKEN = 'my-refresh-token';
```

**Access in Notebook:**

```python
import streamlit as st

token = st.secrets.oauth_token
# Use token for authenticated requests
```

### Managing Secrets

```sql
-- View secrets (values are hidden)
SHOW SECRETS;

-- Describe a secret
DESC SECRET my_api_key;

-- Update a secret
ALTER SECRET my_api_key
  SET SECRET_STRING = 'new-api-key-value';

-- Drop a secret
DROP SECRET my_api_key;

-- Grant usage on secret
GRANT USAGE ON SECRET my_api_key TO ROLE DATA_SCIENTIST;
```

### Secret Best Practices

1. **Never hardcode secrets** in notebook code
2. **Use descriptive names** for secrets
3. **Rotate secrets regularly**
4. **Grant minimal privileges** - only to roles that need them
5. **Use appropriate secret types** for your use case
6. **Document secret purposes** in comments

---

## Troubleshooting

### Common Issues and Solutions

#### 1. Integration Not Visible in Snowsight

**Problem:** Created EAI but can't see it in External Access pane

**Solution:**
```python
# Restart the notebook session
# Click the restart button in Snowsight, or run:
import streamlit as st
st.experimental_rerun()
```

#### 2. Permission Denied Error

**Problem:** `SQL compilation error: Insufficient privileges to operate on integration`

**Solution:**
```sql
-- Ensure role has USAGE privilege
GRANT USAGE ON INTEGRATION pypi_access_integration TO ROLE <your_role>;

-- Verify grants
SHOW GRANTS ON INTEGRATION pypi_access_integration;
```

#### 3. Network Access Denied

**Problem:** `External network access denied`

**Solution:**
```sql
-- Verify network rule includes all required hosts
DESC NETWORK RULE pypi_network_rule;

-- Update if needed
ALTER NETWORK RULE pypi_network_rule
  SET VALUE_LIST = (
    'pypi.org',
    'pypi.python.org',
    'pythonhosted.org',
    'files.pythonhosted.org'
  );
```

#### 4. Secret Not Accessible

**Problem:** `Secret not found` or `KeyError`

**Solution:**
```sql
-- Ensure secret is associated with BOTH EAI AND notebook
ALTER NOTEBOOK my_notebook
  SET EXTERNAL_ACCESS_INTEGRATIONS = (my_integration),
    SECRETS = ('secret_name' = my_secret);

-- Verify the association
DESC NOTEBOOK my_notebook;
```

#### 5. Package Installation Fails

**Problem:** `pip install` fails or times out

**Solution:**
```python
# Use verbose mode to see what's failing
import subprocess
import sys

result = subprocess.run(
    [sys.executable, "-m", "pip", "install", "package_name", "-v"],
    capture_output=True,
    text=True
)
print("STDOUT:", result.stdout)
print("STDERR:", result.stderr)
```

#### 6. Integration Disabled

**Problem:** Integration exists but not working

**Solution:**
```sql
-- Check if integration is enabled
SHOW INTEGRATIONS;

-- Enable if disabled
ALTER INTEGRATION pypi_access_integration SET ENABLED = true;
```

### Debugging Checklist

- [ ] Role has CREATE INTEGRATION privilege
- [ ] Network rule includes all required hosts
- [ ] Integration is ENABLED
- [ ] USAGE privilege granted to correct role
- [ ] Notebook session restarted after EAI creation
- [ ] EAI toggled ON in Snowsight External Access pane
- [ ] Secrets associated with both EAI and notebook (if using secrets)
- [ ] Current role matches the role with USAGE privileges

### Verification Script

Run this in your notebook to verify setup:

```python
import streamlit as st
import subprocess
import sys

print("=== External Access Verification ===")

# Test 1: Check if st.secrets is accessible
try:
    secrets = st.secrets
    print("✓ Secrets accessible")
except Exception as e:
    print(f"✗ Secrets error: {e}")

# Test 2: Try to install a small package
try:
    result = subprocess.run(
        [sys.executable, "-m", "pip", "install", "requests", "--quiet"],
        capture_output=True,
        timeout=30
    )
    if result.returncode == 0:
        print("✓ PyPI access working")
    else:
        print(f"✗ PyPI access failed: {result.stderr.decode()}")
except Exception as e:
    print(f"✗ PyPI access error: {e}")

# Test 3: Check installed packages
try:
    result = subprocess.run(
        [sys.executable, "-m", "pip", "list"],
        capture_output=True,
        text=True
    )
    print(f"\n✓ Installed packages count: {len(result.stdout.splitlines())}")
except Exception as e:
    print(f"✗ Package list error: {e}")
```

---

## Best Practices

### Security

1. **Principle of Least Privilege**
   - Grant USAGE only to roles that need it
   - Don't use PUBLIC role
   - Review grants regularly

2. **Secret Management**
   - Rotate secrets periodically
   - Use appropriate secret types
   - Never log or print secret values
   - Delete unused secrets

3. **Network Rules**
   - Be specific with VALUE_LIST
   - Only include necessary hosts
   - Document why each host is needed
   - Review rules quarterly

### Performance

1. **Package Installation**
   - Install packages once at notebook start
   - Cache installed packages when possible
   - Use specific version numbers
   - Consider creating custom Python environments

2. **API Calls**
   - Implement retry logic
   - Use connection pooling
   - Cache responses when appropriate
   - Set reasonable timeouts

### Organization

1. **Naming Conventions**
   ```sql
   -- Use descriptive, consistent names
   <service>_network_rule      (e.g., pypi_network_rule)
   <service>_access_integration (e.g., pypi_access_integration)
   <service>_<type>_secret     (e.g., openai_api_secret)
   ```

2. **Documentation**
   - Comment SQL statements
   - Document integration purposes
   - Maintain integration inventory
   - Track which notebooks use which EAIs

3. **Version Control**
   - Store EAI creation scripts in Git
   - Document changes to integrations
   - Track secret rotations
   - Maintain deployment procedures

### Maintenance

1. **Regular Reviews**
   - Quarterly: Review all integrations
   - Monthly: Check unused integrations
   - Weekly: Monitor failed access attempts
   - Daily: Check integration health

2. **Monitoring**
   ```sql
   -- Check integration status
   SHOW EXTERNAL ACCESS INTEGRATIONS;
   
   -- Review recent changes
   SHOW PARAMETERS LIKE '%EXTERNAL_ACCESS%' IN ACCOUNT;
   
   -- Audit usage
   SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.ACCESS_HISTORY
   WHERE OBJECTS_MODIFIED LIKE '%INTEGRATION%'
   ORDER BY QUERY_START_TIME DESC
   LIMIT 100;
   ```

---

## Quick Reference

### Essential Commands

```sql
-- Create Network Rule
CREATE NETWORK RULE <name> MODE=EGRESS TYPE=HOST_PORT VALUE_LIST=('host.com');

-- Create EAI
CREATE EXTERNAL ACCESS INTEGRATION <name> ALLOWED_NETWORK_RULES=(<rule>) ENABLED=true;

-- Create Secret
CREATE SECRET <name> TYPE=GENERIC_STRING SECRET_STRING='value';

-- Grant Usage
GRANT USAGE ON INTEGRATION <name> TO ROLE <role>;

-- Associate with Notebook
ALTER NOTEBOOK <name> SET EXTERNAL_ACCESS_INTEGRATIONS=(<integration>);

-- Show Integrations
SHOW EXTERNAL ACCESS INTEGRATIONS;

-- Describe Integration
DESC INTEGRATION <name>;
```

### Python Code Snippets

```python
# Access secret
import streamlit as st
api_key = st.secrets['secret_name']

# Install package
import subprocess, sys
subprocess.check_call([sys.executable, "-m", "pip", "install", "package"])

# Make API call with auth
import requests
response = requests.get(url, headers={'Authorization': f'Bearer {api_key}'})
```

---

## Additional Resources

- **Snowflake Documentation**: [External Access Overview](https://docs.snowflake.com/en/user-guide/ui-snowsight/notebooks-external-access)
- **Snowflake Secrets**: [Creating Secrets](https://docs.snowflake.com/en/sql-reference/sql/create-secret)
- **Network Rules**: [CREATE NETWORK RULE](https://docs.snowflake.com/en/sql-reference/sql/create-network-rule)
- **GitHub Examples**: [Snowflake External Access Examples](https://github.com/Snowflake-Labs/sf-samples/tree/main/samples/external-network-access)

---

## Complete PyPI Setup Example

Here's a complete, copy-paste ready setup for PyPI:

```sql
-- ============================================
-- Complete PyPI External Access Setup
-- ============================================

-- Step 1: Use appropriate role
USE ROLE ACCOUNTADMIN;

-- Step 2: Create network rule
CREATE OR REPLACE NETWORK RULE pypi_network_rule
  MODE = EGRESS
  TYPE = HOST_PORT
  VALUE_LIST = (
    'pypi.org',
    'pypi.python.org',
    'pythonhosted.org',
    'files.pythonhosted.org'
  );

-- Step 3: Create external access integration
CREATE OR REPLACE EXTERNAL ACCESS INTEGRATION pypi_access_integration
  ALLOWED_NETWORK_RULES = (pypi_network_rule)
  ENABLED = true;

-- Step 4: Verify creation
SHOW EXTERNAL ACCESS INTEGRATIONS LIKE 'pypi_access_integration';

-- Step 5: Grant to your role (replace with your role)
GRANT USAGE ON INTEGRATION pypi_access_integration TO ROLE DATA_SCIENTIST;

-- Step 6: Associate with notebook (replace with your notebook name)
ALTER NOTEBOOK my_ml_notebook
  SET EXTERNAL_ACCESS_INTEGRATIONS = (pypi_access_integration);

-- Verification
DESC INTEGRATION pypi_access_integration;
SHOW GRANTS ON INTEGRATION pypi_access_integration;
```

**Test in Notebook:**

```python
# Cell 1: Test PyPI access
import subprocess
import sys

print("Testing PyPI access...")

try:
    result = subprocess.check_call([
        sys.executable, "-m", "pip", "install", 
        "requests", "--quiet"
    ])
    print("✓ SUCCESS: PyPI access is working!")
    print("✓ Package 'requests' installed successfully")
except Exception as e:
    print(f"✗ FAILED: {e}")

# Cell 2: Verify installation
import requests
print(f"✓ requests version: {requests.__version__}")
```

---

*Last Updated: November 12, 2025*
*Based on: [Snowflake Notebooks External Access Documentation](https://docs.snowflake.com/en/user-guide/ui-snowsight/notebooks-external-access)*

