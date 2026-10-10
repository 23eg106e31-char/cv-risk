# CV Risk AI — NHANES + TabNet Full Stack

A React + Spring Boot + FastAPI web application for cardiovascular risk stratification using **your supplied NHANES dataset** (`ml-service/data/nhanes_cvd_crp_combined_harmonized.csv`).

## Dataset integrated
- 17,041 rows
- 50 columns
- Model target: `major_cvd_target`
- 14 prediction inputs are used to avoid outcome/target leakage: age, sex, BMI, systolic/diastolic BP, total cholesterol, HDL, LDL, triglycerides, glucose, HbA1c, hs-CRP, current smoking, diabetes.
- Missing numeric values are median-imputed using training data.
- NHANES categorical fields are harmonized automatically.

## 1. Train and start ML service
```bash
cd ml-service
python -m venv .venv
# Windows
.venv\\Scripts\\activate
# macOS/Linux
# source .venv/bin/activate
pip install -r requirements.txt
python src/train.py --data_path data/nhanes_cvd_crp_combined_harmonized.csv --max_epochs 100 --patience 15
uvicorn app:app --host 0.0.0.0 --port 8000
```

Training creates `ml-service/models/` and `ml-service/outputs/`.

## 2. Start Spring Boot
Open another terminal:
```bash
cd backend
mvn spring-boot:run
```
Backend: http://localhost:8080

## 3. Start React
Open another terminal:
```bash
cd frontend
npm install
npm run dev
```
Frontend: http://localhost:5173

## Important
This ZIP now includes a trained `ml-service/models/preprocessor.joblib` and a bundled `ml-service/models/fallback_model.joblib`. The API uses the real TabNet model when `tabnet_cv_risk.zip` is present, otherwise it automatically uses the bundled fallback model. This prevents the previous assessment failure caused by missing model artifacts.

The `/health` endpoint reports the active model type. Feature explanation errors are also isolated so they cannot turn a valid prediction into an HTTP 500.

This is an educational/research prototype, not a medical diagnostic tool.


### Fix for 422 missing-field error
The ML API now accepts the three legacy fields `hypertensionTreated`, `familyHistoryCvd`, and `physicalActivityScore` with defaults, so older frontend/backend builds will not fail validation. Restart all three services after extracting this updated ZIP.
