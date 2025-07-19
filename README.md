# Deforestation Analysis API

This project provides a simple Node.js server that acts as a proxy to the Sentinel Hub API, allowing users to analyze deforestation in a selected geographic area.

The API fetches satellite imagery from two different time periods (the last month and the month prior), compares the Normalized Difference Vegetation Index (NDVI), and identifies potential deforestation hotspots.

---

## 1. Project Setup

Before running the application, you need to set up your environment variables.

### **Prerequisites**

* [Node.js](https://nodejs.org/) installed on your system.
* A Sentinel Hub account to get your `CLIENT_ID` and `CLIENT_SECRET`. You can create a free account for testing purposes.

### **Configuration**

1.  **Clone the project** or save the `server.js` file to your local machine.
2.  **Install dependencies:** This project uses `dotenv` to manage environment variables. Run the following command in your project directory:
    ```bash
    npm install dotenv
    ```
3.  **Create a `.env` file** in the same directory as `server.js`.
4.  **Add your Sentinel Hub credentials** to the `.env` file like this:

    ```
    CLIENT_ID="your_sentinel_hub_client_id"
    CLIENT_SECRET="your_sentinel_hub_client_secret"
    ```

### **Running the Server**

Start the server by running the following command in your terminal:

```bash
node server.js
```

If successful, you will see the following message:

```
🚀 Server running at http://localhost:3000
Endpoint available: POST /analyze-deforestation
```

---

## 2. API Usage with `curl`

You can interact with the API using any HTTP client, but this guide provides an example using `curl`.

### **Endpoint: `/analyze-deforestation`**

* **URL:** `https://carboneye.onrender.com/analyze-deforestation`
* **Method:** `POST`
* **Description:** Analyzes a specified geographic area for vegetation loss.

### **Example `curl` Command**

This command analyzes a region in the state of Rondônia, Brazil, which has experienced significant deforestation.

```bash
curl -X POST https://carboneye.onrender.com/analyze-deforestation \
-H "Content-Type: application/json" \
-d '{"bbox": [-62.88, -10.35, -62.38, -9.85]}'
```

**Payload Details:**

The `-d` flag sends the JSON data payload. The `bbox` (bounding box) is an array of four geographic coordinates in the following order:

`[minimum longitude, minimum latitude, maximum longitude, maximum latitude]`

---

## 3. Understanding the API Response

The server will respond with a detailed JSON object containing the analysis results.

### **Example Response Structure**

```json
{
  "today": {
    "trueColor": "data:image/png;base64,...",
    "ndvi": "data:image/png;base64,..."
  },
  "past": {
    "trueColor": "data:image/png;base64,...",
    "ndvi": "data:image/png;base64,..."
  },
  "alerts": [
    {
      "position": {
        "lat": -10.12345,
        "lon": -62.56789
      },
      "severity": "critical",
      "change": "-0.451"
    },
    {
      "position": {
        "lat": -10.23456,
        "lon": -62.67890
      },
      "severity": "moderate",
      "change": "-0.223"
    }
  ],
  "analysis": {
    "totalAlerts": 2,
    "criticalAlerts": 1,
    "moderateAlerts": 1,
    "timeRange": {
      "recent": "2025-06-16 to 2025-07-16",
      "past": "2025-05-16 to 2025-06-16"
    }
  }
}
```

### **Response Fields:**

* `today`: An object containing base64-encoded strings for the **True Color** and **NDVI** images from the last 30 days.
* `past`: An object containing the same image types from the 30 days prior.
* `alerts`: An array of JSON objects, where each object represents a detected deforestation hotspot.
    * **`position`**: This is the most important field for mapping. It contains the **latitude (`lat`) and longitude (`lon`)** of the alert.
    * `severity`: `critical` or `moderate`, based on the degree of vegetation loss.
    * `change`: The calculated negative change in the NDVI value.
* `analysis`: A summary object containing the total number of alerts and the time ranges used for the comparison.
