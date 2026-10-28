# Data Versioning using DVC (Data Version Control) & Git

This repository demonstrates how to implement **Data Version Control (DVC)** alongside **Git** to track, version, and roll back datasets in Machine Learning / MLOps pipelines.

---

## 📌 Prerequisites & Tools Used

* **Python 3.x** (with `pandas` installed)
* **Git** (for source code version control)
* **DVC (Data Version Control)** (for dataset versioning)
* **VS Code / Terminal**

---

## 📁 Repository Structure

```text
.
├── .dvc/                  # Internal DVC metadata, cache, and config
├── .dvcignore             # Specifies files/folders ignored by DVC
├── .gitignore             # Git ignore file (includes data/ folder)
├── data/                  # Local data folder (ignored by Git, managed by DVC)
│   └── sample_data.csv    # Generated dataset
├── data.dvc                # DVC pointer file storing dataset MD5 hash
├── my_code.py             # Python script generating/updating data
├── project_flow.txt       # Step-by-step workflow tracking
├── s3/                    # Local folder simulating remote S3 storage
└── README.md              # Project documentation
```

---

## 🚀 Step-by-Step Walkthrough

### **Step 1: Setup GitHub Repository & Clone Locally**
1. Create a new public repository on GitHub (e.g., `mlops-dvc-data-versioning`).
2. Clone the repository to your local machine and navigate into the project directory:
   ```bash
   git clone <YOUR_GITHUB_REPO_URL>
   cd mlops-dvc-data-versioning
   ```

---

### **Step 2: Create Python Data Generator Script (`my_code.py`)**
Create `my_code.py` to generate initial sample data (`data/sample_data.csv` with 3 initial records):

```python
import os
import pandas as pd

# Initial Dataset (Version 1)
data = {
    'Name': ['Alice', 'Bob', 'Charlie'],
    'Age': [25, 30, 35],
    'City': ['CityA', 'CityB', 'CityC']
}

df = pd.DataFrame(data)

# Ensure data directory exists
data_dir = 'data'
os.makedirs(data_dir, exist_ok=True)

# Save DataFrame to CSV
file_path = os.path.join(data_dir, 'sample_data.csv')
df.to_csv(file_path, index=False)
print("Data generated successfully at:", file_path)
```

Run the script to create the initial dataset:
```bash
python my_code.py
```

---

### **Step 3: Initial Git Commit (Before DVC)**
Commit the initial source code to Git before introducing DVC:
```bash
git add .
git commit -m "First commit before DVC"
git push origin main
```

---

### **Step 4: Initialize DVC & Configure Remote Storage**
1. Initialize DVC in your project repository:
   ```bash
   dvc init
   ```
2. Create a local folder to simulate AWS S3 remote storage:
   ```bash
   mkdir s3
   ```
3. Register the local `s3` folder as the default DVC remote storage endpoint:
   ```bash
   dvc remote add -d myremote s3
   ```

---

### **Step 5: Hand Data Tracking over from Git to DVC**
1. Attempting to add data directly to DVC (`dvc add data`) raises an error if Git is already tracking it.
2. Remove the `data/` directory from Git tracking:
   ```bash
   git rm -r --cached data
   git commit -m "Stop tracking data in Git"
   ```
3. Start tracking the `data/` folder with DVC:
   ```bash
   dvc add data
   ```
   * *What happens under the hood:*
     * Generates `data.dvc` containing a unique **MD5 hash pointer**.
     * Automatically adds `data/` to `.gitignore`.

---

### **Step 6: Save & Push Data Version 1 (v1)**
1. Track DVC metadata (`data.dvc` and `.gitignore`) in Git:
   ```bash
   git add data.dvc .gitignore
   git commit -m "DVC init with data v1"
   ```
2. Push raw data files to the remote storage (`s3`):
   ```bash
   dvc commit
   dvc push
   ```
3. Commit Git state and push to remote GitHub:
   ```bash
   git add .
   git commit -m "First version of data saved"
   git push origin main
   ```

---

### **Step 7: Update Data to Version 2 (v2)**
1. Modify `my_code.py` to append a new record (4th row):
   ```python
   # Append 4th row (Version 2)
   new_row = {'Name': 'GF1', 'Age': 20, 'City': 'City1'}
   df = pd.concat([df, pd.DataFrame([new_row])], ignore_index=True)
   df.to_csv(file_path, index=False)
   ```
2. Re-run `my_code.py` and inspect status:
   ```bash
   python my_code.py
   dvc status
   ```
3. Save Version 2 in DVC and Git:
   ```bash
   dvc commit
   dvc push
   git add my_code.py data.dvc
   git commit -m "Second version of data saved"
   git push origin main
   ```

---

### **Step 8: Update Data to Version 3 (v3)**
1. Modify `my_code.py` again to append a 5th record:
   ```python
   # Append 5th row (Version 3)
   new_row_2 = {'Name': 'GF2', 'Age': 28, 'City': 'City2'}
   df = pd.concat([df, pd.DataFrame([new_row_2])], ignore_index=True)
   df.to_csv(file_path, index=False)
   ```
2. Re-run `my_code.py`, commit, and push Version 3:
   ```bash
   python my_code.py
   dvc commit
   dvc push
   git add my_code.py data.dvc
   git commit -m "Third version of data saved"
   git push origin main
   ```

---

### **Step 9: Rolling Back & Synchronizing Data Versions**

#### **A. Roll back to Data Version 1 (3 records)**
1. Check Git commit history to locate the commit hash for Version 1:
   ```bash
   git log --oneline
   ```
2. Checkout the Git commit corresponding to Version 1:
   ```bash
   git checkout <COMMIT_HASH_V1>
   ```
3. Check DVC status (DVC detects that local data files do not match the checked-out `.dvc` hash):
   ```bash
   dvc status
   ```
4. Restore matching dataset version from DVC cache:
   ```bash
   dvc checkout
   ```
   *Inspect `data/sample_data.csv` — it is instantly swapped back to 3 rows!*

#### **B. Restore back to Latest Main Branch (Version 3)**
1. Switch Git back to the `main` branch:
   ```bash
   git checkout main
   ```
2. Pull/checkout latest matching dataset from DVC:
   ```bash
   dvc pull
   ```
   *Inspect `data/sample_data.csv` — it restores back to 5 rows!*

---

## 💡 Key Takeaways

1. **Git + DVC Separation**: Git handles code and tiny `.dvc` pointer files; DVC manages raw datasets and model artifacts.
2. **MD5 Hash Tracking**: Every time dataset content changes, DVC computes a new MD5 hash and updates `data.dvc`.
3. **Seamless Rollbacks**: Rolling back code via Git restores `.dvc` pointer files; running `dvc checkout` / `dvc pull` aligns local dataset files instantly.
