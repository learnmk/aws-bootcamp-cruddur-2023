# Week 2 — Distributed Tracing

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

![Screenshot 2023-03-05 at 11 20 52 AM](https://user-images.githubusercontent.com/125124581/223090128-fc85296e-b1ac-419e-936c-2edca9a093fd.png)

![Screenshot 2023-03-05 at 11 59 24 AM](https://user-images.githubusercontent.com/125124581/223090233-c7569a81-7743-4b6a-9423-a9a75e1d219d.png)


### TASK 3
Install WatchTower and write a custom logger to send application log data to - CloudWatch Log group

![Screenshot 2023-03-05 at 1 12 29 PM](https://user-images.githubusercontent.com/125124581/223089987-91db726a-56d5-4921-88a8-eb6a38315fa6.png)


### TASK 4 
Integrate Rollbar for Error Logging. Trigger an error an observe an error with Rollbar


![Screenshot 2023-03-05 at 2 33 17 PM](https://user-images.githubusercontent.com/125124581/223089870-0d984174-c18a-4dbf-b235-e4b96d94eceb.png)
