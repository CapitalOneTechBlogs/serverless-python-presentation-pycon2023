---
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Best Practices for Using Python to Power Serverless Applications
  ### PyCon 2023 Presentation Materials

# persist drawings in exports and build
drawings:
  persist: true
# page transition
transition: slide-left
# use UnoCSS
css: unocss
---

# Best Practices for Using Python to Power Serverless Applications

## Capital One

PyCon 2023

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    Press Space for next page <carbon:arrow-right class="inline"/>
  </span>
</div>

<div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button>
  <a href="https://github.com/mcnamarabrian/pycon2023" target="_blank" alt="GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---
transition: fade-out
layout: two-cols
---

::right::

Dan Furman

<img alt="Dan's Profile Picture" src="https://avatars.githubusercontent.com/u/4566402?v=4" width=200 />

- [<logos-twitter />@djfurman4tech](https://twitter.com/djfurman4tech)
- [<logos-linkedin-icon />:/djfurman](https://linkedin.com/in/djfurman)

Distinguished Engineer

Card Tech, Data

::default::

Brian McNamara

<img alt="Brian's Profile Picture" src="https://avatars.githubusercontent.com/u/17259?v=4" width=200 />

- [<logos-twitter />@mcnamarabrian](https://twitter.com/mcnamarabrian)
- [<logos-linkedin-icon />:/mcnamarabrian](https://linkedin.com/in/mcnamarabrian)

Distinguished Engineer

Retail Bank, Serverless

<!-- Introduce ourselves
    Years developing Python
    Years developing Serverless
    What do we do for Capital One

    We'll make this look better, but this gives us a starting point

    Thank you for taking the time to learn more about how Capital One uses Python to power a large number of serverless applications.
-->

---
transition: fade-out
---

# Why Python for Serverless?

- Simple is better than complex
- Fast development cycle
- Fantastic scaling
- Lower cost

<!--
    Establish why

    - Python is the FIRST choice for modern serverless apps
    - You should consider building serverless apps
    - Cost is based on usage; pay for what you actually consume
      - Total cost of ownership
-->

---
transition: slide-down
---

# Kinds of Serverless Applications

- API provider
- Streaming data processor
- Machine Learning Inference

<!--
    Emphasize flexibility of Python in Serverless

    - All of these are Event driven!
    - Code as Glue
    - Logical organization of functions/dependencies
    - Handle processing when needed
-->

---
transition: slide-down
---

# API Application

<img src="/api-arch.png" width=400 />

<!--
    Build logic into Lambda function (compute layer)

    - Stateless functions work best
    - Interact with other services as needed hosted by your cloud or self-managed installs
-->

---
transition: slide-down
---

# Streaming Data Application

<img src="/stream-arch.png" width=400 />

<!--
    Do your transformations on stream in real time

    - Use basic compute functions
    - Or work directly with PySpark/Ray via AWS Glue
-->

---
transition: slide-down
---

# ML Inference Application

<img src="/ml-arch.png" width=400 />

<!--
    Deploy your inference code

    - Release your model to EFS as a shared mount point
    - Each model update is a release of a managed assset!
-->

---

# The Serverless Mindset

<div v-click>
- Code is ephemeral
</div>
<div v-click>
- Code is isolated
</div>
<div v-click>
- Code is self-contained
</div>
<div v-click>
- Code scales without intervention
</div>

<!--
    This requires some trade offs!
    When everything is an event, so are your logging, observability, and deployments
-->

---

# Key Questions

<div v-click>
- How do you observe your application (e.g., logs, metrics, performance)?
</div>
<div v-click>
- How do you test, build, and deploy your application?
</div>
<div v-click>
- How do you architect your application?
</div>
<div v-click>
- Should you use Lambda for your application?
</div>

<!-- Great questions to answer; want to add mental maps (concept not necessarily diagrams) to existing patterns -->

---
transition: slide-up
---

# How do you know what your Lambda application is doing?

- Rules of observability change
- Logs, metrics, and traces all must persist outside of the function

---
transition: slide-up
layout: image-right
image: https://source.unsplash.com/collection/94734566/1920x1080
---
## Observability Pro Tips

<div v-click>
- Structure your logs - JSON is nice
</div>

<div v-click>
- Set a retention period on your logs
</div>

<div v-click>
- Log what you need but allow for debugging if necessary
</div>

<div v-click>
- Emit custom metrics asynchronously using <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Embedded_Metric_Format_Specification.html">Embedded Metrics Format (EMF)</a>
</div>

<div v-click>
- Understand the different types of Lambda metrics and the statistics that should be used with each
</div>

<div v-click>
- <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/using-metric-math.html">Metric math functions</a> are available to you
</div>

<div v-click>
- Don't just look at the average - look at the edges (p90, p99)
</div>

<div v-click>
...just use <a href="https://awslabs.github.io/aws-lambda-powertools-python/latest">AWS Lambda Powertools</a>
</div>

<!--
    Edges drive your customer experience
    JSON searchable can be sent to any log aggregator
-->

---
transition: slide-up
---
# Logging

Sample of using AWS Lambda Powertools package for logging[^1] and metrics collection[^2]

```python
from aws_lambda_powertools import Logger, Metrics
from aws_lambda_powertools.metrics import MetricUnit
from aws_lambda_powertools.utilities.typing import LambdaContext

logger = Logger()
metrics = Metrics()


@logger.inject_lambda_context
@metrics.log_metrics
def handler(event: dict, context: LambdaContext) -> str:
    logger.info("Collecting payment")

    logger.info({"operation": "charge", "charge_id": event["charge_id"]})
    metrics.add_metric(name="SuccessfulCharge", unit=MetricUnit.Count, value=event["charge_amount"])
    return {
      "operation": "charge",
      "charge_id": event["charge_id"],
      "charge_amount": event["charge_amount"]
    }
```

<arrow v-click="3" x1="400" y1="420" x2="230" y2="330" color="#564" width="3" arrowSize="1" />

[^1]: [Logging](https://awslabs.github.io/aws-lambda-powertools-python/latest/core/logger/)
[^2]: [Metrics](https://awslabs.github.io/aws-lambda-powertools-python/latest/core/metrics/)

<style>
.footnotes-sep {
  @apply mt-20 opacity-10;
}
.footnotes {
  @apply text-sm opacity-75;
}
.footnote-backref {
  display: none;
}
</style>

---

# Tracing Pro Tips

- Use Powertools to seamlessly
  - Annotate your functions
  - Instrument your packages
  - Inject keys for your telemetry

---

# Trace your code

```python
from uuid import uuid4
from aws_lambda_powertools import Tracer

# Automatically instrument calls to external dependencies
tracer = Tracer(patch_modules=["requests"])


@tracer.capture_method
def generate_uuid4() -> str:
    # You can apply this at any level to understand what is needed
    return str(uuid4())


@tracer.capture_method
def post_payment(user_id: str) -> dict:
    tracer.put_annotation(key="user_id", value=user_id)
    # Process the payment here
```

<!--
    Tracing provides your telemetry
    Output any keys needed for processing
-->

---

# Live Code

These examples are nice; but we're at PyCon! Let's do some real code.

Join us at [github.com/mcnamarabrian/pycon203](https://github.com/mcnamarabrian/pycon2023)

<!--
    Switch to live code example and walk thorugh code patterns
-->

---

# Thank you for joining us!

<!--
    Can discuss audience questions as time allows
 -->
