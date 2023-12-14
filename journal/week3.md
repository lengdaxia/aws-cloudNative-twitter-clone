# Week 3 — Decentralized Authentication

# X-Ray

### Instrument AWS X-Ray for Flask

```
export AWS_REGION="ap-northeast-1"
gp env AWS_REGION="ap-northeast-1"
```

Add to the `Requirements.txt`
```
aws-xray-sdk
```

Install python dependencies
```
pip install -r requirements.txt
```

Add to `app.py`
```
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware

xray_url = os.getenv("AWS_XRAY_URL")
xray_recorder.configure(service='Cruddur', dynamic_naming=xray_url)
XRayMiddleware(app, xray_recorder)
```

Setup AWS X-Ray Resources
Add `aws/json/xray.json`
```
{
  "SmaplingRule": {
    "RuleName": "Cruddur",
    "ResourceARN": "*",
    "Priority": 9000,
    "FixedRate": 0.1,
     "ReservoirSize": 5,
      "ServiceName": "Cruddur",
      "ServiceType": "*",
      "Host": "*",
      "HTTPMethod": "*",
      "URLPath": "*",
      "Version": 1
  }
}
```
```sh
FLASK_ADDRESS="https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
aws xray create-group \
   --group-name "Cruddur" \
   --filter-expression "service(\"$FLASK_ADDRESS\") {fault OR error}"
```

```
aws xray create-sampling-rule --cli-input-json file://aws/json/xray.json
```

### Add Deamon Service To Docker Compose
```
  xray-daemon:
    image: "amazon/aws-xray-daemon"
    environment:
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
      AWS_REGION: "us-east-1"
    command:
      - "xray -o -b xray-daemon:2000"
    ports:
      - 2000:2000/udp
```

We need to add these two env vars to our backend-flask in our docker-compose.yml file

```
      AWS_XRAY_URL: "*4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}*"
      AWS_XRAY_DAEMON_ADDRESS: "xray-daemon:2000"
```

### Check service data for last 10 minutes
```
EPOCH=$(date +%s)
aws xray get-service-graph --start-time $(($EPOCH-600)) --end-time $EPOCH
```

### HoneyComb
---

### CloudWatch Log
---

### Rollbar
---

### [Note] Changes to Rollbar
---
