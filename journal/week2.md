# Week 2 — Distributed Tracing

## What is the need for Distributed Tracing system
1 Trace user requests through your application while meeting your security and compliance objectives.
2 Identify bottlenecks and determine where high latencies are occurring to improve application performance.
3 Remove data silos and get the information you need to improve user experience and reduce downtime.
4 Debug serverless applications in real time, and monitor both cloud cost and performance metrics.


### TASK 1 
Instrumented backend flask application to use Open Telemetry (OTEL) with Honeycomb.io as the provider. 

## Installing Honeycomb

On the backend-flask/requirements.text, add required python libraries and install dependancies.

```py
opentelemetry-api 
opentelemetry-sdk 
opentelemetry-exporter-otlp-proto-http 
opentelemetry-instrumentation-flask 
opentelemetry-instrumentation-requests
```

```sh
pip install -r requirements.txt
```

Update app.py configuration in backend-flask app with below code.

```py
# Honeycomb
from opentelemetry import trace
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

# Honeycomb
# Initialize tracing and an exporter that can send data to Honeycomb
provider = TracerProvider()
processor = BatchSpanProcessor(OTLPSpanExporter())
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)

# Honeycomb
# Initialize automatic instrumentation with Flask
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()
```

Add Enviroment variable in docker-compose.yml
```yml
OTEL_SERVICE_NAME: 'backend-flask'
OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
```

Login into your honecomb.io account and copy API Key add key into Gitpod environment variable.

```
gp env HONEYCOMB_API_KEY=""
```

We are creating span and attribute for that need to updated code in home_activities.py

```py
from opentelemetry import trace
tracer = trace.get_tracer("home.activities")

with tracer.start_as_current_span("home-activities-mock-data"):
    span = trace.get_current_span()
    
span.set_attribute("app.now", now.isoformat())

span.set_attribute("app.result_lenght", len(results))
```

Output of the queries run to explore traces within Honeycomb.io

![Screenshot 2023-03-06 at 4 25 52 PM](https://user-images.githubusercontent.com/125124581/223091310-6c65455c-5e96-4c7f-bca5-7391534f3bf4.png)


### TASK 2
Integrated AWS X-Ray into backend flask application. Configure and provisioned X-Ray daemon within docker-compose and send data back to X-Ray API. Observed X-Ray traces within the AWS Console. 

## Install Xray

On the backend-flask/requirements.text, add required python libraries and install dependancies.
```txt
aws-xray-sdk
```
```sh
pip install -r requirements.txt
```

Create xray.json inside the folder of /aws/json/ with below code

```json
{
    "SamplingRule": {
        "RuleName": "Cruddur",
        "ResourceARN": "*",
        "Priority": 9000,
        "FixedRate": 0.1,
        "ReservoirSize": 5,
        "ServiceName": "backend-flask",
        "ServiceType": "*",
        "Host": "*",
        "HTTPMethod": "*",
        "URLPath": "*",
        "Version": 1
    }
  }
```

Update app.py configuration in backed flask as below

```py
# Xray
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware

# Xray
xray_url = os.getenv("AWS_XRAY_URL")
xray_recorder.configure(service='backend-flask', dynamic_naming=xray_url)
```

Create Group in Xray with below AWS cli command

```sh
aws xray create-group --group-name "Cruddur" --filter-expression "service(\"backend-flask")"
```

Update docker-compose.yaml with following code

```docker
AWS_XRAY_URL: "*4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}*"
AWS_XRAY_DAEMON_ADDRESS: "xray-daemon:2000"

xray-daemon:
    image: "amazon/aws-xray-daemon"
    environment:
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
      AWS_REGION: "eu-west-2"
    command:
      - "xray -o -b xray-daemon:2000"
    ports:
      - 2000:2000/udp
 ```
 
 ```
simple_processor = SimpleSpanProcessor(ConsoleSpanExporter())
provider.add_span_processor(simple_processor)
```
```
# xray
XRayMiddleware(app, xray_recorder)
```

AWS Xray Screenshot.

![Screenshot 2023-03-05 at 11 20 52 AM](https://user-images.githubusercontent.com/125124581/223090128-fc85296e-b1ac-419e-936c-2edca9a093fd.png)

Subsegment configuration screenshot

![Screenshot 2023-03-05 at 11 59 24 AM](https://user-images.githubusercontent.com/125124581/223090233-c7569a81-7743-4b6a-9423-a9a75e1d219d.png)


### TASK 3
Install WatchTower and write a custom logger to send application log data to - CloudWatch Log group

On the backend-flask/requirements.text, add required python libraries and install dependancies.
```
watchtower
opentelemetry-instrumentation-requests
```
```sh
pip install -r requirements.txt
```

Update app.py for backend-flask 

```py
# Cloudwatch
import watchtower
import logging
from time import strftime

# Configuring Logger to Use CloudWatch
LOGGER = logging.getLogger(__name__)
LOGGER.setLevel(logging.DEBUG)
console_handler = logging.StreamHandler()
cw_handler = watchtower.CloudWatchLogHandler(log_group='cruddur')
LOGGER.addHandler(console_handler)
LOGGER.addHandler(cw_handler)
LOGGER.info("test log")

@app.after_request
def after_request(response):
    timestamp = strftime('[%Y-%b-%d %H:%M]')
    LOGGER.error('%s %s %s %s %s %s', timestamp, request.remote_addr, request.method, request.scheme, request.full_path, response.status)
    return response
```

Update home_activities.py 
```py
def run(Logger):
   LOGGER.info("HomeActivities")
```

Add below environment variable in docker-compose.yaml

```
AWS_DEFAULT_REGION: "${AWS_DEFAULT_REGION}"
AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
```

Cloudwatch logs implementation screenshot.

![Screenshot 2023-03-05 at 1 12 29 PM](https://user-images.githubusercontent.com/125124581/223089987-91db726a-56d5-4921-88a8-eb6a38315fa6.png)


### TASK 4 
Integrate Rollbar for Error Logging. Trigger an error an observe an error with Rollbar

On the backend-flask/requirements.text, add required python libraries and install dependancies.
```
blinker
rollbar
```

```sh
pip install -r requirements.txt
```
Login into Rollbar account and grabbed Rollbar access token

Add env in Gitpod

```
export ROLLBAR_ACCESS_TOKEN=""
gp env ROLLBAR_ACCESS_TOKEN=""
```

In docker-compose.yaml

```
ROLLBAR_ACCESS_TOKEN: "${ROLLBAR_ACCESS_TOKEN}"
```

For Integrating Rollbar with App update app.py

```py
import rollbar
import rollbar.contrib.flask
from flask import got_request_exception

# Rollbar
rollbar_access_token = os.getenv('ROLLBAR_ACCESS_TOKEN')
@app.before_first_request
def init_rollbar():
    """init rollbar module"""
    rollbar.init(
        # access token
        rollbar_access_token,
        # environment name
        'production',
        # server root directory, makes tracebacks prettier
        root=os.path.dirname(os.path.realpath(__file__)),
        # flask already sets up logging
        allow_logging_basic_config=False)

    # send exceptions from `app` to rollbar, using flask's signal system.
    got_request_exception.connect(rollbar.contrib.flask.report_exception, app)

@app.route('/rollbar/test')
def rollbar_test():
    rollbar.report_message('Hello World!', 'warning')
    return "Hello World!"
```

Screenshot of error response in Rollbar

![Screenshot 2023-03-05 at 2 33 17 PM](https://user-images.githubusercontent.com/125124581/223089870-0d984174-c18a-4dbf-b235-e4b96d94eceb.png)


# Homework Challege

## Task 1: Adding custom span with userid 

Added below code in home_activities.py

```py
handle = results[0]['handle']
span.set_attribute("app.user_id", handle)
```
Screenshot of span in Honecomb.

![Screenshot 2023-03-06 at 5 45 52 PM](https://user-images.githubusercontent.com/125124581/223110068-3ef4a7b6-a560-4030-9e76-52f9b57d429e.png)

## Task 2: Run Custom Queries in Honeycomb

Run custom query to check latency by userid

![Screenshot 2023-03-06 at 6 05 41 PM](https://user-images.githubusercontent.com/125124581/223112092-447f9a5f-c028-4d0d-b4aa-db6675cffc22.png)




