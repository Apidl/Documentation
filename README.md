# APIDL API Documentation (v1)
> **Transform any API into an Intelligent Agent instantly.**
>
> APIDL bridges the gap between natural language and backend logic. It empowers developers to turn standard RESTful services into autonomous AI Agents capable of understanding context, chaining complex workflows, and executing tasks on behalf of users—without writing rigid, rule-based code.

## Introduction
APIDL (API Directive Language & AI Prompt Integration Definition Language) allows developers to integrate context-aware AI orchestration into their applications. Unlike standard LLM endpoints, APIDL is designed to execute tasks. By passing dynamic context (via `custom_params`), you enable the AI to understand the user's specific state (e.g., auth tokens, user IDs, location) and perform actions on their behalf.

**Base URL:** `https://v1.apidl.org`

---

## Authentication
Authentication is enforced via the `api_key` parameter included in the JSON request body.

> **Security Warning:** Your API Key is a secret credential.
> * **Server-side:** Safe to use directly.
> * **Client-side (Web/Mobile/IoT):** It is recommended to route requests through your own proxy server to keep the key hidden, or use strict CORS policies if allowed.

---

## Endpoint: Run Agent

Processes a natural language prompt, retrieves conversation history based on the unique ID, and orchestrates necessary API calls using the provided custom parameters.

* **URL:** `/run`
* **Method:** `POST`
* **Content-Type:** `application/json`

### Request Parameters

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `api_key` | `string` | **Yes** | Your unique API secret key. |
| `message` | `string` | **Yes** | The natural language instruction, question, or command from the user. |
| `uniq_id` | `string` | **Yes** | A unique identifier for the end-user (e.g., `user_123`, `device_mac_addr`). This is required to maintain **conversation history** and context continuity. |
| `custom_params`| `object` | **No** | Key-value pairs containing dynamic context required for API orchestration. See details below. |

### Understanding `custom_params`
The `custom_params` object serves as the "bridge" between the AI and your backend data. If the AI needs to fetch private data (e.g., "Show my invoices"), it requires the user's credentials.

**Example Scenario:**
User asks: *"What is my balance?"*
The AI needs the `user_id` and `access_token` to call your internal API. You provide these here.

```json
"custom_params": {
    "user_id": 4501,
    "access_token": "eyJh...",
    "device_type": "mobile_android",
    "location": "Istanbul, TR"
}

```
**Success Response (200 OK)**
```json
{
  "status": "success",
  "data": {
    "response": "I've checked your account. Your current balance is $1,250.00.",
    "conversation_id": "conv_88123"
  }
}
```

**Error Response**
```json
{
  "status": "error",
  "code": "invalid_api_key",
  "message": "The provided API key is incorrect."
}
```

## Code Examples

**cURL (Terminal):**
```bash
curl -X POST [https://v1.apidl.org/run](https://v1.apidl.org/run) \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "YOUR_API_KEY",
    "message": "Update my profile name to John Doe",
    "uniq_id": "user_session_123",
    "custom_params": {
      "auth_token": "bearer_token_xyz",
      "user_id": 99
    }
  }'
  
  ```

### Advanced: Multi-Step Chained Workflow

APIDL excels at complex orchestration where the output of one action is required for the next.

**Scenario:** An e-commerce manager wants to create a product, add options, and set a discount in a single sentence.
* **Step 1:** AI creates the product -> *System returns new Product ID (e.g., #5501)*.
* **Step 2:** AI uses ID #5501 to add color variations.
* **Step 3:** AI uses ID #5501 to apply a special price/discount rule.

**cURL Request:**

```bash
curl -X POST [https://v1.apidl.org/run](https://v1.apidl.org/run) \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "YOUR_ADMIN_API_KEY",
    "message": "Create a new product named \"Titan Gaming Mouse\". Add Red, Blue, and Green color variations. Finally, apply a 15% discount to this product.",
    "uniq_id": "admin_session_99",
    "custom_params": {
        "admin_token": "secure_write_token_xyz",
        "shop_id": 105,
        "currency": "USD",
        "default_stock": 100
    }
  }'
```

**JavaScript (Fetch API - Web/Node.js)**
```javascript
const runAgent = async () => {
  const payload = {
    api_key: "YOUR_API_KEY",
    message: "Do I have any pending tasks?",
    uniq_id: "user_browser_fingerprint_88",
    custom_params: {
      token: localStorage.getItem("site_token"),
      current_page: window.location.pathname
    }
  };

  try {
    const response = await fetch("[https://v1.apidl.org/run](https://v1.apidl.org/run)", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(payload)
    });

    const result = await response.json();
    console.log(result.data.response);
  } catch (error) {
    console.error("Error:", error);
  }
};

runAgent();
```

**PHP (cURL)**
```php
<?php
$url = '[https://v1.apidl.org/run](https://v1.apidl.org/run)';
$data = [
    'api_key' => 'YOUR_API_KEY',
    'message' => 'Generate a report for last month sales.',
    'uniq_id' => 'user_id_55',
    'custom_params' => [
        'admin_level' => 1,
        'shop_id' => 102
    ]
];

$ch = curl_init($url);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($data));
curl_setopt($ch, CURLOPT_HTTPHEADER, ['Content-Type: application/json']);

$response = curl_exec($ch);
curl_close($ch);

$result = json_decode($response, true);
echo $result['data']['response'];
?>
```

**Python (Requests)**
```python
import requests
import json

url = "[https://v1.apidl.org/run](https://v1.apidl.org/run)"
payload = {
    "api_key": "YOUR_API_KEY",
    "message": "Find the nearest service center.",
    "uniq_id": "session_py_99",
    "custom_params": {
        "lat": 41.0082,
        "lon": 28.9784
    }
}

headers = {"Content-Type": "application/json"}

try:
    response = requests.post(url, json=payload, headers=headers)
    response.raise_for_status()
    data = response.json()
    print(data["data"]["response"])
except requests.exceptions.RequestException as e:
    print(f"Error: {e}")
    
    ```

**Android (Kotlin - using Retrofit/OkHttp)**
```kotlin
// Define data class for request
data class AgentRequest(
    val api_key: String,
    val message: String,
    val uniq_id: String,
    val custom_params: Map<String, Any>
)

// Inside your Coroutine or Thread
val client = OkHttpClient()
val jsonMediaType = "application/json; charset=utf-8".toMediaType()

val requestBody = AgentRequest(
    api_key = "YOUR_API_KEY",
    message = "Turn on the living room lights.",
    uniq_id = "android_device_id_uuid",
    custom_params = mapOf("home_id" to 12, "access_code" to "abc-123")
)

val jsonPayload = Gson().toJson(requestBody)
val body = jsonPayload.toRequestBody(jsonMediaType)

val request = Request.Builder()
    .url("[https://v1.apidl.org/run](https://v1.apidl.org/run)")
    .post(body)
    .build()

client.newCall(request).execute().use { response ->
    if (!response.isSuccessful) throw IOException("Unexpected code $response")
    println(response.body?.string())
}
```

**iOS (Swift)**
```swift
import Foundation

let url = URL(string: "[https://v1.apidl.org/run](https://v1.apidl.org/run)")!
var request = URLRequest(url: url)
request.httpMethod = "POST"
request.setValue("application/json", forHTTPHeaderField: "Content-Type")

let parameters: [String: Any] = [
    "api_key": "YOUR_API_KEY",
    "message": "Check my flight status.",
    "uniq_id": "ios_user_identifier",
    "custom_params": [
        "flight_pnr": "PNR123",
        "last_name": "Doe"
    ]
]

request.httpBody = try? JSONSerialization.data(withJSONObject: parameters)

let task = URLSession.shared.dataTask(with: request) { data, response, error in
    guard let data = data, error == nil else {
        print(error?.localizedDescription ?? "No data")
        return
    }
    
    if let responseJSON = try? JSONSerialization.jsonObject(with: data, options: []) {
        print(responseJSON)
    }
}

task.resume()
```

**IoT / Embedded (C++ for ESP32/Arduino)**
```C++
#include <WiFi.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>

const char* api_url = "[https://v1.apidl.org/run](https://v1.apidl.org/run)";

void sendToAgent(String userMessage) {
  if(WiFi.status() == WL_CONNECTED) {
    HTTPClient http;
    http.begin(api_url);
    http.addHeader("Content-Type", "application/json");

    // Create JSON Payload
    StaticJsonDocument<512> doc;
    doc["api_key"] = "YOUR_API_KEY";
    doc["message"] = userMessage;
    doc["uniq_id"] = "iot_device_mac_address";
    
    JsonObject custom_params = doc.createNestedObject("custom_params");
    custom_params["sensor_temp"] = 24.5;
    custom_params["sensor_humidity"] = 60;
    
    String requestBody;
    serializeJson(doc, requestBody);

    // Send POST
    int httpResponseCode = http.POST(requestBody);

    if(httpResponseCode > 0) {
      String response = http.getString();
      Serial.println(httpResponseCode);
      Serial.println(response);
    } else {
      Serial.print("Error on sending POST: ");
      Serial.println(httpResponseCode);
    }
    http.end();
  }
}
```

## Unleash Unlimited Potential

With APIDL, you aren't just sending requests; you are giving your application a brain. The true power lies in the flexibility of your own API services.

We designed APIDL to be agnostic. It doesn't matter if you are building a complex e-commerce manager, a smart home controller, or a data analysis tool. By exposing modular and flexible API endpoints, you enable the AI to mix, match, and orchestrate them in ways you might not have even planned manually.

* **Build flexible services.**
* **Let the AI orchestrate.**
* **Scale your logic instantly.**

If you can build an API for it, APIDL can turn it into an intelligent agent. **The only limit is your imagination.**
