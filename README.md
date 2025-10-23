# Customer Churn Prediction API utilizing Vertex AI

A machine learning API for predicting customer churn using scikit-learn models deployed on Google Cloud Platform. This service provides both single customer a FastAPI-based REST API.

## 🎯 What This Project Does

This project provides a machine learning API that:

- **Predicts customer churn** using a trained scikit-learn model

- **Integrates with Google Cloud Platform** for model storage and deployment
- **Provides confidence scores** and detailed prediction metadata

## 🏗️ What Has Been Implemented in project.ipynb

The notebook demonstrates a Google Cloud ML pipeline for customer churn prediction:

### **1. Google Cloud Platform Setup**
- **Project Configuration**: Sets up GCP project ID, region (us-central1), and storage bucket
- **Service Initialization**: Configures Google Cloud Storage client and AI Platform
- **Bucket Management**: Creates and manages `cloud-office-ml-bucket` for storing model artifacts

### **2. Data Processing Pipeline**
- **Data Loading**: Reads telecom customer churn dataset from Google Cloud Storage (previously uploaded as a .csv file)
- **Preprocessing**: Implements a custom `Preprocessor` class that:
  - Cleans column names (lowercase, underscore replacement)
  - Handles categorical features (standardizes string values)
  - Processes numerical features (converts totalcharges to numeric, fills missing values)
  - Transforms target variable (churn: Yes/No → 1/0)

### **3. Machine Learning Model Training**
- **Model Selection**: Uses Logistic Regression with liblinear solver
- **Feature Vectorization**: Implements DictVectorizer for categorical feature encoding
- **Model Training**: Trains the model on preprocessed customer data
- **Model Persistence**: Saves trained model and preprocessor using joblib and pickle

### **4. Model Artifact Management**
- **Storage**: Uploads model artifacts (model.joblib, preprocessor.pkl) to Google Cloud Storage
- **Organization**: Stores artifacts in structured directory (`coml-artifact-dir`)
- **Versioning**: Maintains model versions in cloud storage for deployment

### **5. FastAPI Application Development**
- **API Structure**: Creates REST API with health check and prediction endpoints
- **Data Models**: Implements Pydantic models for request/response validation
- **Model Integration**: Downloads and loads model artifacts from GCS at startup
- **Prediction Logic**: Implements churn prediction with probability scoring

### **6. Container Deployment**
- **Dockerfile Creation**: Builds optimized Docker container with:
  - Python 3.12-slim base image
  - uv package manager for dependency management
  - FastAPI and uvicorn for web serving
  - Google Cloud client libraries
- **Container Registry**: Pushes Docker image to Google Artifact Registry
- **Image Tagging**: Uses proper naming convention for container images

### **7. Google Cloud AI Platform Integration**
- **Model Upload**: Registers trained model with AI Platform
- **Container Deployment**: Deploys custom container to AI Platform endpoints
- **Service Configuration**: Sets up machine types
- **Endpoint Management**: Creates and manages prediction endpoints

### **8. Testing and Validation**
- **Test Client**: Creates Python script for API testing
- **Sample Data**: Generates realistic customer data for prediction testing
- **API Validation**: Tests both health check and prediction endpoints
- **Response Verification**: Validates prediction accuracy and response format

### **9. Production Deployment**
- **Cloud Build**: Uses Google Cloud Build for automated container building
- **Artifact Registry**: Manages Docker images in Google Artifact Registry
- **AI Platform Endpoints**: Deploys model to scalable prediction endpoints
- **Resource Management**: Configures appropriate machine types and scaling

### **Key Features Implemented:**
- **End-to-end ML pipeline** from data preprocessing to production deployment
- **Scalable containerized deployment** using Docker and Google Cloud
- **REST API** with proper request/response handling
- **Model versioning and artifact management** in cloud storage
- **Production-ready infrastructure** with proper monitoring and health checks

## 🚀 How to Deploy and Run

### Prerequisites

- Python 3.12+
- Docker
- Google Cloud SDK
- Google Cloud Project with billing enabled

### Required Google Cloud APIs

The following APIs need to be enabled for this project:

| API Name | Purpose |
|----------|---------|
| `aiplatform.googleapis.com` | Vertex AI API |
| `artifactregistry.googleapis.com` | Artifact Registry API |
| `bigquery.googleapis.com` | BigQuery API |
| `cloudbuild.googleapis.com` | Cloud Build API |
| `cloudresourcemanager.googleapis.com` | Cloud Resource Manager API |
| `compute.googleapis.com` | Compute Engine API |
| `containerregistry.googleapis.com` | Container Registry API |
| `iam.googleapis.com` | Identity and Access Management API |
| `logging.googleapis.com` | Cloud Logging API |
| `run.googleapis.com` | Cloud Run Admin API |
| `storage-api.googleapis.com` | Google Cloud Storage JSON API | 

### Local Development

1. **Clone and setup**:
   ```bash
   git clone <repository-url>
   cd cloud-office-ml
   ```

### Google Cloud Platform Deployment

1. **Enable required APIs**:
   ```bash
   gcloud services enable cloudbuild.googleapis.com
   gcloud services enable aiplatform.googleapis.com
   gcloud services enable storage.googleapis.com
   ```

2. **Create service account**:
   ```bash
   gcloud iam service-accounts create cloud-office-ml-sa \
     --display-name="Cloud Office ML Service Account"
   
   gcloud projects add-iam-policy-binding $(gcloud config get-value project) \
     --member="serviceAccount:cloud-office-ml-sa@$(gcloud config get-value project).iam.gserviceaccount.com" \
     --role="roles/storage.objectViewer"
   ```

3. **Build and push to Google Container Registry**:
   ```bash
   gcloud builds submit --tag gcr.io/$(gcloud config get-value project)/cloud-office-ml .
   ```

4. **Deploy to Cloud Run**:
   ```bash
   gcloud run deploy cloud-office-ml \
     --image gcr.io/$(gcloud config get-value project)/cloud-office-ml \
     --service-account cloud-office-ml-sa@$(gcloud config get-value project).iam.gserviceaccount.com \
     --region us-central1 \
     --platform managed \
     --allow-unauthenticated
   ```

## 📊 API Endpoints

### Health Check
- **GET** `/health` - Service health status

### Predictions
- **POST** `/predict` - Single customer churn prediction

### Example Request
```json
{
  "gender": "female",
  "seniorcitizen": 0,
  "partner": "yes",
  "dependents": "no",
  "phoneservice": "no",
  "multiplelines": "no_phone_service",
  "internetservice": "dsl",
  "onlinesecurity": "no",
  "onlinebackup": "yes",
  "deviceprotection": "no",
  "techsupport": "no",
  "streamingtv": "no",
  "streamingmovies": "no",
  "contract": "month-to-month",
  "paperlessbilling": "yes",
  "paymentmethod": "electronic_check",
  "tenure": 1,
  "monthlycharges": 29.85,
  "totalcharges": 29.85
}
```

### Example Response
```json
{
  "churn_probability": 0.7234,
  "churn": true
}
```

## 🔧 Configuration

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `GOOGLE_CLOUD_PROJECT` | GCP Project ID | Required |
| `BUCKET_NAME` | GCS bucket for model artifacts | `cloud-office-ml-bucket` |
| `MODEL_ARTIFACT_DIR` | Directory in bucket for artifacts | `coml-artifact-dir` |
| `AIP_HEALTH_ROUTE` | Health check endpoint | `/health` |
| `AIP_PREDICT_ROUTE` | Prediction endpoint | `/predict` |
| `PROBABILITY_THRESHOLD` | Churn probability threshold | `0.5` |

## 📈 What I'd Improve With More Time

### Testing components
- **Unit testing** for components

### Monitoring and Observability
- **Structured logging** with correlation IDs
- **Distributed tracing** with OpenTelemetry
- **Alerting** for prediction accuracy drift

### Security Enhancements
- **API rate limiting** to prevent abuse
- **Audit logging** for compliance

### Data Quality
- **Data validation** schemas
- **Automated data pipeline** testing
