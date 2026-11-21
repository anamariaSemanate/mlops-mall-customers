# MLOps Mall Customers

Proyecto MLOps completo basado en el tutorial *"Cómo construir un proyecto MLOps completo con Python, MLflow, FastAPI, Streamlit y Grafana"* (Juan Manuel Núñez Velasco).

Clasifica clientes del dataset **Mall Customers** en 4 segmentos:

| Segmento | Descripción |
|---|---|
| 0 | Cliente premium |
| 1 | Cliente conservador |
| 2 | Cliente joven con alto gasto |
| 3 | Cliente de bajo valor |

**Stack:** Python · Scikit-learn · MLflow · FastAPI · Streamlit · Prometheus · Grafana · Docker · GitHub Actions

## Estructura

```
mlops-mall-customers/
├── .github/workflows/ci.yml   # CI con GitHub Actions
├── api/main.py                # API FastAPI (/predict, /health, /metrics)
├── app/streamlit_app.py       # Frontend Streamlit
├── data/
│   ├── raw/Mall_Customers.csv # Dataset (ya incluido)
│   ├── processed/             # Datos limpios y segmentados
│   └── monitoring/            # Log de predicciones de la API
├── docker/prometheus.yml      # Configuración de Prometheus
├── reports/
│   ├── figures/               # Codo, silhouette, PCA, matrices de confusión
│   └── metrics/               # Resumen de segmentos y reportes
├── src/
│   ├── config.py              # Configuración centralizada
│   ├── data.py                # Carga y limpieza
│   ├── features.py            # Columnas y escalado
│   ├── clustering.py          # KMeans + PCA
│   ├── train.py               # Entrenamiento + tracking en MLflow
│   ├── register_model.py      # Registro del mejor modelo (alias champion)
│   └── predict.py             # Predicción local
├── tests/test_api.py          # Tests unitarios
├── docker-compose.yml         # Prometheus + Grafana
├── pytest.ini
└── requirements.txt
```

## Cómo ejecutarlo (Windows / PowerShell)

### 1. Entorno virtual e instalación
```powershell
cd mlops-mall-customers
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```
> Si PowerShell bloquea la activación: `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`

### 2. Preprocesamiento y segmentación
```powershell
python -m src.data
python -m src.clustering
```

### 3. MLflow (Terminal 1, dejarla abierta)
```powershell
mlflow server --host 127.0.0.1 --port 5000
```
Abrir http://127.0.0.1:5000

### 4. Entrenar, registrar y predecir (Terminal 2, con el venv activo)
```powershell
python -m src.train
python -m src.register_model
python -m src.predict
```

### 5. API FastAPI (Terminal 2)
```powershell
uvicorn api.main:app --reload --host 127.0.0.1 --port 8000
```
Abrir http://127.0.0.1:8000/docs y probar `/predict` con:
```json
{ "gender": "Male", "age": 28, "annual_income_k": 60, "spending_score": 80 }
```

### 6. Streamlit (Terminal 3)
```powershell
streamlit run app/streamlit_app.py
```
Abrir http://localhost:8501

### 7. Prometheus + Grafana (requiere Docker Desktop abierto)
```powershell
docker compose up -d
```
- Prometheus: http://localhost:9090 → consultar `mall_customer_prediction_count_total`
- Grafana: http://localhost:3000 (usuario `admin` / contraseña `admin`)
  - Agregar data source Prometheus con URL `http://prometheus:9090`
  - Paneles sugeridos:
    - `mall_customer_prediction_count_total`
    - `rate(mall_customer_prediction_count_total[1m])`
    - `mall_customer_segment_predictions_total`

### 8. Tests
```powershell
pytest
```

### 9. Subir a GitHub
```powershell
git init
git add .
git commit -m "Proyecto MLOps Mall Customers"
git remote add origin https://github.com/TU_USUARIO/mlops-mall-customers.git
git branch -M main
git push -u origin main
```
El workflow `.github/workflows/ci.yml` se ejecutará automáticamente en cada push a `main`/`develop`.

## Ajustes respecto al tutorial original

- **`src/train.py`**: se añadió `skops_trusted_types=["sklearn.tree._tree.Tree"]` en `log_model`. Las versiones recientes de MLflow (3.x) rechazan por defecto guardar Random Forest y Gradient Boosting sin este parámetro.
- **`pytest.ini`**: agrega la raíz del proyecto al path para que `pytest` encuentre el paquete `api` (sin esto los tests fallan con `ModuleNotFoundError`).
- **`.gitignore`**: excluye `.venv`, `mlruns/`, `mlartifacts/` y archivos generados, pero **sí** incluye `data/raw/Mall_Customers.csv`, que necesita GitHub Actions.
- En el Paso 14 del tutorial hay un error tipográfico (`vicorn`); el comando correcto es `uvicorn`.
