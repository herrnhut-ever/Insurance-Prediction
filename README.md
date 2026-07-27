# Insurance Charge Prediction Web App

A machine learning web application that predicts medical insurance charges based on customer attributes. Built with **FastAPI** and **PyCaret**, this project demonstrates best practices for deploying ML models in production.

## 🎯 Features

- **Web Form Interface**: User-friendly form to input customer data and get predictions
- **JSON API**: Programmatic access via REST endpoints for third-party integration
- **Interactive API Docs**: Auto-generated Swagger UI at `/docs`
- **Production-Ready**: Implements proper MLOps patterns (model loading at startup, lifespan management)
- **Health Checks**: Liveness/readiness endpoints for container orchestration
- **Containerized**: Includes Dockerfile for easy deployment

## 📁 Project Structure

```
.
├── app.py                          # FastAPI application
├── Insurance - Model Training Notebook.ipynb  # Model training notebook
├── requirements.txt                # Python dependencies
├── Dockerfile                      # Docker configuration
├── Procfile                        # Render deployment configuration
├── DEPLOYMENT.md                   # Deployment guide
├── static/
│   └── style.css                   # CSS styling
└── templates/
    └── home.html                   # HTML form interface
```

## 🚀 Quick Start

### Prerequisites

- Python 3.8+
- pip

### Installation

1. Clone the repository:
```bash
git clone https://github.com/yourusername/Insurance-Prediction.git
cd Insurance-Prediction
```

2. Install dependencies:
```bash
pip install -r requirements.txt
```

3. Run the application:
```bash
python app.py
```

The application will start at `http://localhost:8000`

## 📊 API Endpoints

### Web Interface
- **GET** `/` - Home page with prediction form

### Form Submission
- **POST** `/predict` - Submit form data to get prediction
  - Parameters: `age`, `sex`, `bmi`, `children`, `smoker`, `region`

### JSON API
- **POST** `/predict_api` - Get prediction via JSON
  ```json
  {
    "age": 19,
    "sex": "female",
    "bmi": 27.9,
    "children": 0,
    "smoker": "yes",
    "region": "southwest"
  }
  ```

### Health Check
- **GET** `/health` - Service status and model availability

## 🤖 Model Training

The model is trained using PyCaret's regression module. To retrain:

1. Open `Insurance - Model Training Notebook.ipynb` in Jupyter
2. Run all cells to train and export the model
3. The trained model is saved as `deployment_28042020`

## 🐳 Deployment

### Docker

Build and run with Docker:
```bash
docker build -t insurance-predictor .
docker run -p 8000:8000 insurance-predictor
```

### Cloud Deployment

See [DEPLOYMENT.md](DEPLOYMENT.md) for instructions on deploying to Render, AWS, or other platforms.

## 📦 Requirements

See [requirements.txt](requirements.txt) for the complete list of dependencies, which includes:
- FastAPI
- Uvicorn
- PyCaret
- Pandas
- Pydantic

## 📚 Learning Resources

- [PyCaret Official Website](https://www.pycaret.org)
- [FastAPI Documentation](https://fastapi.tiangolo.com)
- [Original Medium Post](https://medium.com/@moez_62905/build-and-deploy-your-first-machine-learning-web-app-280c53d3800a)

## 💡 MLOps Best Practices Demonstrated

- **Startup/Shutdown Hooks**: Model loaded once at startup, not per-request
- **Type Validation**: Pydantic models for automatic request validation
- **Separation of Concerns**: Shared `run_prediction()` logic for both web form and API
- **Health Checks**: Meaningful status endpoint for orchestration platforms
- **Environment Configuration**: PORT via environment variables

## 📝 License

This project is based on the PyCaret beginner's tutorial.

## 🤝 Contributing

Contributions are welcome! Feel free to open issues or submit pull requests.
