# mlfs-book
O'Reilly book - Building Machine Learning Systems with a feature store: batch, real-time, and LLMs

Fredrik Ström - frest@kth.se
Group: Fred

The project implements a complete ML pipeline:

1. Backfill Feature Pipeline (Notebook 1)
	•	Downloads >1 year of historical weather data from Open-Meteo.
	•	Loads historical PM2.5 data (CSV) from AQICN.
	•	Computes lagged PM2.5 features for each sensor:
	•	pm25_lag_1d
	•	pm25_lag_2d
	•	pm25_lag_3d
	•	Registers two Feature Groups in Hopsworks:
	•	weather
	•	air_quality_munich

---

2. Daily Feature Pipeline (Notebook 2)

Runs once per day via GitHub Actions or Modal:
	•	Fetches yesterday’s air-quality (PM2.5) for all Munich sensors
	•	Fetches yesterday’s weather
	•	Fetches 7-day weather forecast
	•	Computes lagged PM2.5 using the most recent 3 data points per station
	•	Inserts updated data into the Feature Store

---

3. Training Pipeline (Notebook 2)
	•	Creates a Feature View combining:
	•	Weather features
	•	PM2.5 + lagged features
	•	station_id
	•	Splits training/validation by date (no shuffle)
	•	Trains an XGBoost Regressor
	•	Registers the model in Hopsworks

---

4. Batch Inference + Dashboard (Notebook 4 – Grade E)
	•	Loads model and upcoming weather forecast
	•	Performs autoregressive prediction for next 7 days:
	•	Uses updated lagged PM2.5 for each forecast step
	•	Writes predictions to a monitoring Feature Group
	•	Creates:
	•	Forecast plot (next 7 days)
	•	Hindcast plot (prediction vs reality)

---

5. Prediction Monitoring (Grade E)
	•	Hindcast aligns:
	•	Actual PM2.5
	•	Predicted PM2.5
	•	Allows performance monitoring over time

--- 

The model was extended to include:
	•	pm25_lag_1d
	•	pm25_lag_2d
	•	pm25_lag_3d

These significantly improved stability and predictive quality.

However, some sensors show extreme PM2.5 spikes (outliers), which increased MSE, particularly when training on multiple sensors

---

I decided to do a multi-sensor support for Munich. I queried:

`https://api.waqi.info/search/?keyword=munich&token=...` 

to extract all Munich station UIDs. Each daily run pulls PM2.5 for every station. 
Weather is retrieved once per day and cross-joined to each sensor.

Each station has independent:
	•	pm25_lag_1d
	•	pm25_lag_2d
	•	pm25_lag_3d

To support multiple sensors, the following features were added: 

	•	station_id (logged as float64 to match Hopsworks)
	•	per-sensor lagged PM2.5

---

Per-sensor forecast and hindcast dashboards

Plots are saved for:
	•	all sensors combined
	•	each individual sensor

