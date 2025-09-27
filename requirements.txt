from fastapi import FastAPI, Path ,HTTPException, Query
import json
from fastapi.responses import JSONResponse
from schema.user_input import UserInput
import pandas as pd
from model.predict import model, MODEL_VERSION, predict_output
from schema.prediction_response import PredictionResponse

app = FastAPI()
        
@app.get('/')
def read_root():
    return {"message": "Hey there! Welcome to the Insurance Premium Prediction API."}


@app.get('/health')
def health_check():
    return {
        "status": "ok",
        "version": MODEL_VERSION,
        "model": model is not None
        }

@app.post('/predict' , response_model = PredictionResponse)
def predict_premium(data: UserInput):
    
    user_input= {
        'bmi': data.bmi,
        'age_group': data.age_group,
        'lifestyle_risk': data.lifestyle_risk,
        'city_tier': data.city_tier,
        'income_lpa': data.income_lpa,
        'occupation': data.occupation
    }


    try:
        prediction = predict_output(user_input)
        return JSONResponse(status_code=200, content={'predicted_category': prediction})
    except Exception as e:
        raise HTTPException(status_code=500, content = str(e))
