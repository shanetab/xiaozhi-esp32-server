# Context Provider Usage Tutorial

## Overview

A "Context Provider" adds data sources to the context of prompts for the Xiaozhi system.

A "Context Provider" obtains data from external systems when the Xiaozhi system is awakened, and dynamically injects it into the large model's system prompt (System Prompt).

This allows Xiaozhi to perceive the state of certain things in the world at the moment of awakening.

It is fundamentally different from MCP and memory: 
- "Context Provider" forcibly makes Xiaozhi perceive world data; 
- "Memory (Mem)" makes it know what was previously discussed; 
- "MCP (function call)" is used when needing to invoke specific capabilities/knowledge.

With this feature, Xiaozhi can perceive at the moment of awakening:
- Health sensor status (body temperature, blood pressure, blood oxygen status, etc.)
- Real-time data from business systems (server load, pending data, stock information, etc.)
- Any textual information that can be obtained via HTTP API

**Note**: This feature is only for helping Xiaozhi perceive the state of things at the moment of awakening. If you want Xiaozhi to continuously perceive the state of things after awakening, it's recommended to combine this feature with MCP tool calls.

## Working Principle

1. **Configure Source**: Users configure one or more HTTP API addresses.
2. **Trigger Request**: When the system builds the Prompt, if it detects the `{{ dynamic_context }}` placeholder in the template, it will request all configured APIs.
3. **Automatic Injection**: The system will automatically format the API's returned data into a Markdown list and replace the `{{ dynamic_context }}` placeholder.

## Interface Specification

To enable Xiaozhi to correctly parse the data, your API needs to meet the following specifications:

- **Request Method**: `GET`
- **Request Header**: The system will automatically add the `device-id` field to the Request Header.
- **Response Format**: Must return JSON format containing `code` and `data` fields.

### Response Examples

**Case 1: Return key-value pairs**
```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "Living Room Temperature": "26℃",
    "Living Room Humidity": "45%",
    "Front Door Status": "Closed"
  }
}
```
*Injection effect:*
```markdown
<context>
- **Living Room Temperature:** 26℃
- **Living Room Humidity:** 45%
- **Front Door Status:** Closed
</context>
```

**Case 2: Return list**
```json
{
  "code": 0,
  "data": [
    "You have 10 pending tasks",
    "Current car speed is 100 km/h"
  ]
}
```
*Injection effect:*
```markdown
<context>
- You have 10 pending tasks
- Current car speed is 100 km/h
</context>
```

## Configuration Guide

### Method 1: Smart Console Configuration (Full Module Deployment)

1. Log into the Smart Console and go to the **Role Configuration** page.
2. Find the **Context Provider** configuration item (click the "Edit Source" button).
3. Click **Add**, and enter your API address.
4. If the API requires authentication, you can add `Authorization` or other Headers in the **Request Headers** section.
5. Save the configuration.

### Method 2: Configuration File Configuration (Single Module Deployment)

Edit the `xiaozhi-server/data/.config.yaml` file, add the `context_providers` configuration section:

```yaml
# Context Provider Configuration
context_providers:
  - url: "http://api.example.com/data"
    headers:
      Authorization: "Bearer your-token"
  - url: "http://another-api.com/data"
```

## Enabling the Feature

By default, the system's prompt template file (`data/.agent-base-prompt.txt`) already contains the `{{ dynamic_context }}` placeholder, so you don't need to add it manually.

**Example:**

```markdown
<context>
[Important! The following information has been provided in real-time, no tool queries needed, use directly:]
- **Device ID:** {{device_id}}
- **Current Time:** {{current_time}}
...
{{ dynamic_context }}
</context>
```

**Note**: If you don't need this feature, you can choose **not to configure any context providers**, or **remove** the `{{ dynamic_context }}` placeholder from the prompt template file.

## Appendix: Mock Test Service Example

For your convenience in testing and development, we provide a simple Python Mock Server script. You can run this script to simulate an API interface locally.

**mock_api_server.py**

```python
import http.server
import socketserver
import json
from urllib.parse import urlparse, parse_qs

# Set port number
PORT = 8081

class MockRequestHandler(http.server.SimpleHTTPRequestHandler):
    def do_GET(self):
        # Parse path and parameters
        parsed_path = urlparse(self.path)
        path = parsed_path.path
        query = parse_qs(parsed_path.query)

        response_data = {}
        status_code = 200

        print(f"Received request: {path}, parameters: {query}")

        # Case 1: Simulate health data (return dictionary Dict)
        # Path parameter style: /health
        # device_id from Header
        if path == "/health":
            device_id = self.headers.get("device-id", "unknown_device")
            print(f"device_id: {device_id}")
            response_data = {
                "code": 0,
                "msg": "success",
                "data": {
                    "Test Device ID": device_id,
                    "Heart Rate": "80 bpm",
                    "Blood Pressure": "120/80 mmHg",
                    "Status": "Good"
                }
            }

        # Case 2: Simulate news list (return list List)
        # No parameters: /news/list
        elif path == "/news/list":
            response_data = {
                "code": 0,
                "msg": "success",
                "data": [
                    "Top News: Python 3.14 Released",
                    "Tech News: AI Assistants Changing Lives",
                    "Local News: Heavy Rain Tomorrow, Remember to Bring an Umbrella"
                ]
            }

        # Case 3: Simulate weather briefing (return string String)
        # No parameters: /weather/simple
        elif path == "/weather/simple":
            response_data = {
                "code": 0,
                "msg": "success",
                "data": "Sunny to cloudy today, temperatures 20-25°C, air quality excellent, suitable for outdoor activities."
            }

        # Case 4: Simulate device details (Query parameter style)
        # Parameter style: /device/info
        # device_id from Header
        elif path == "/device/info":
            device_id = self.headers.get("device-id", "unknown_device")
            response_data = {
                "code": 0,
                "msg": "success",
                "data": {
                    "Query Method": "Header parameter",
                    "Device ID": device_id,
                    "Battery": "85%",
                    "Firmware": "v2.0.1"
                }
            }
        
        # Case 5: 404 Not Found
        else:
            status_code = 404
            response_data = {"error": "Interface does not exist"}

        # Send response
        self.send_response(status_code)
        self.send_header('Content-type', 'application/json; charset=utf-8')
        self.end_headers()
        self.wfile.write(json.dumps(response_data, ensure_ascii=False).encode('utf-8'))

# Start service
# Allow address reuse to prevent errors on quick restarts
socketserver.TCPServer.allow_reuse_address = True
with socketserver.TCPServer(("", PORT), MockRequestHandler) as httpd:
    print(f"==================================================")
    print(f"Mock API Server Started: http://localhost:{PORT}")
    print(f"Available interfaces:")
    print(f"1. [Dict] http://localhost:{PORT}/health")
    print(f"2. [List] http://localhost:{PORT}/news/list")
    print(f"3. [String] http://localhost:{PORT}/weather/simple")
    print(f"4. [Parameter] http://localhost:{PORT}/device/info")
    print(f"==================================================")
    try:
        httpd.serve_forever()
    except KeyboardInterrupt:
        print("\nServer stopped")
```

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.