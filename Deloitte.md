## Q1: When defining a pipeline, what libraries from the SDK do you use?  
### The core is the kfp package. The main imports:  
from kfp import dsl  
from kfp import compiler  
from kfp import Client  
from kfp import dsl  
from kfp.dsl import Input, Output, Dataset, Model  

kfp.dsl — the domain-specific language for authoring. This is where the key decorators and constructs live:  

@dsl.component — turns a Python function into a pipeline component  
@dsl.pipeline — defines the pipeline itself  
dsl.Condition / dsl.If — conditional execution  
dsl.ParallelFor — fan-out loops  
dsl.Input, dsl.Output, and artifact types (Dataset, Model, Metrics, Artifact) for typed data passing between steps   

kfp.compiler.Compiler — compiles the pipeline into IR YAML (in KFP v2) for submission.   
kfp.Client — connects to the KFP API server to upload, run, and manage pipelines and experiments programmatically.  

## Q2: What is the use of NumPy?
NumPy (Numerical Python) is the foundational library for numerical computing in Python. 
Its core contribution is the ndarray — an N-dimensional array — plus the operations around it.  

## Q3: Given an array with one duplicate and one missing number, how would you find them?  
```
def find_dup_missing(arr):
    n = len(arr)
    count = [0] * (n + 1)
    for x in arr:
        count[x] += 1
    dup = miss = -1
    for i in range(1, n + 1):
        if count[i] == 2:
            dup = i
        elif count[i] == 0:
            miss = i
    return dup, miss
```

## Q4: How would you check if a number is a power of 2?
```
def is_power_of_two(n):
    if n <= 0:
        return False
    while n % 2 == 0:
        n //= 2
    return n == 1
```

## Q5: What is dsl.component?
@dsl.component is the KFP decorator that turns a plain Python function into a pipeline component — a single, self-contained, containerized step in your pipeline.  

## Q6: What is dsl.pipeline?
@dsl.pipeline is the decorator that defines the pipeline itself — the function that wires individual components together into a DAG (directed acyclic graph) describing the whole workflow.

## Q7: What is dsl.Condition?
dsl.Condition is the KFP construct for conditional execution inside a pipeline — it runs a block of steps only if a condition evaluates to true at runtime.

## Q8: How do you create and compile a Kubeflow pipeline?
Step 1 — Define the components (@dsl.component)  
Step 2 — Define the pipeline (@dsl.pipeline) — wire components into the DAG  
Step 3 — Compile to IR YAML  
Step 4 — (Submit/run)   
> The mental summary for the interview:
  Write components as containerized Python functions → wire them into a DAG in the pipeline function →
  compile that pipeline function into IR YAML → submit the YAML to the cluster.  
The thing to emphasize: you compile the @dsl.pipeline function, not the components. The components are pulled in automatically because the pipeline function references them.
The output is a single YAML containing all component specs plus the DAG.  

## Q9: How do you submit/run a pipeline in Kubeflow?
Once you have the compiled IR YAML, there are a few ways to run it — the choice depends on whether it's interactive, one-off, or automated.  
Option 1 — Programmatically via kfp.Client (most common, and CI-friendly)  
Option 2 — Upload a reusable pipeline, then trigger runs  
Option 3 — Recurring / scheduled runs (e.g. nightly retraining)  
Option 4 — The UI — upload the YAML in the Kubeflow Pipelines dashboard, create an experiment, click "Create run," fill in parameters. 
Fine for ad-hoc/manual work, not for automation.  

**What happens after you submit (the flow):**
```
client → KFP API server → stores metadata (MySQL + MLMD)
                        → submits workflow to Argo
                        → Argo schedules each component as a pod
                        → status streams back to client
```

