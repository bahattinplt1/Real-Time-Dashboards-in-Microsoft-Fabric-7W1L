# 📊 Microsoft Fabric Real-Time Dashboard

This project demonstrates how to build a real-time dashboard using **Microsoft Fabric**, **Eventhouse**, **Eventstream**, and **Kusto Query Language (KQL)**. The dashboard visualizes live data, supports interactive filtering, and auto-refreshes periodically.

> 🕒 Estimated time to complete: ~25 minutes  
> 🔐 Requirement: A Microsoft Fabric tenant with Trial, Premium, or Fabric capacity enabled.

---

## 🛠️ Steps Overview

### 1. Create a Workspace
- Visit [Microsoft Fabric](https://app.fabric.microsoft.com/home?experience=fabric)
- Create a new workspace with Fabric capacity enabled (Trial, Premium, or Fabric)

### 2. Create an Eventhouse
- From the **Create** menu, select **Eventhouse**
- Assign a unique name
- Open the automatically created **KQL database**

### 3. Create an Eventstream
- In the KQL database, select **Get Data** > **Eventstream** > **New Eventstream**
- Name: `Bicycle-data`
- Select **Use sample data**
  - Source name: `Bicycles`
  - Sample: `Bicycles`
- Set destination settings:
  - Destination: **Eventhouse**
  - Table name: `bikes`
  - Ingestion mode: `Event processing before ingestion`
  - Input format: `JSON`
- Connect the source node to the destination and click **Publish**

### 4. Create a Real-Time Dashboard
- Create a new **Real-Time Dashboard**: `bikes-dashboard`
- Add a new **OneLake data hub** source:
  - Display name: `Bike Rental Data`
  - Database: your Eventhouse DB
  - Passthrough identity: selected

#### Tile 1: Stacked Bar Chart
```kql
bikes
| where ingestion_time() between (ago(30min) .. now())
| summarize latest_observation = arg_max(ingestion_time(), *) by Neighbourhood
| project Neighbourhood, latest_observation, No_Bikes, No_Empty_Docks
| order by Neighbourhood asc
```

#### Tile 2: Map Visualization
```kql
bikes
| where ingestion_time() between (ago(30min) .. now())
| summarize latest_observation = arg_max(ingestion_time(), *) by Neighbourhood
| project Neighbourhood, latest_observation, Latitude, Longitude, No_Bikes
| order by Neighbourhood asc
```

### 5. Create a Base Query
- Variable name: `base_bike_data`
```kql
bikes
| where ingestion_time() between (ago(30min) .. now())
| summarize latest_observation = arg_max(ingestion_time(), *) by Neighbourhood
```
- Update visual queries to use `base_bike_data`

### 6. Add a Parameter
- Add a multiple selection parameter:
  - Label: `Neighbourhood`
  - Variable: `selected_neighbourhoods`
  - Type: `string`
  - Data source: `Bike Rental Data`
```kql
bikes
| distinct Neighbourhood
| order by Neighbourhood asc
```
- Update base query to:
```kql
bikes
| where ingestion_time() between (ago(30min) .. now())
  and (isempty(['selected_neighbourhoods']) or Neighbourhood in (['selected_neighbourhoods']))
| summarize latest_observation = arg_max(ingestion_time(), *) by Neighbourhood
```

### 7. Add a Second Page
- Add a new page: `Page 2`
- Add a new tile with:
```kql
base_bike_data
| project Neighbourhood, latest_observation
| order by latest_observation desc
```

### 8. Configure Auto Refresh
- Enable auto-refresh
  - Minimum interval: Allow all
  - Default refresh rate: **30 minutes**

### 9. Save and Share
- Save the dashboard
- Click **Share > Copy link**
- Open the link in a new tab to test sharing functionality

---

## 📁 Folder Structure
```
microsoft-fabric-realtime-dashboard/
├── README.md
└── screenshots/
    ├── step1-workspace.png
    ├── step2-eventhouse.png
    ├── ...
```

---

## 🖼️ Screenshots

Screenshots for each step are stored in the `screenshots/` folder. Sample names:
- `screenshots/step1-workspace.png`
- `screenshots/step2-eventhouse.png`
- `screenshots/step3-eventstream.png`

Embed them in markdown if needed:
```markdown
![Step 1](screenshots/step1-workspace.png)
```

---

## 📌 Notes
- Real-time data can take a minute or more to appear
- Parameters allow dynamic filtering without modifying queries
- Dashboards are accessible via shared links (login required)

---
