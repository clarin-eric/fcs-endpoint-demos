# FCS Endpoint Demos

This repository contains various demo FCS endpoint implementations. Each implementation will focus on a specific topic and tries to be as concise as possible so that readers can easily recognize what is crucial for the implementation.

## Implementations

### [AAI](aai/)

This endpoint demonstrates how to enable AAI so that Resources with availability restrictions can be announced and access allowed or denied based on authentication info being sent from FCS clients (Aggregator).

The `web.xml` shows needed configurations while the SearchEngine implementation will highlight how to access the authentication info to further process search requests for restricted Resources. The `endpoint-description.xml` will show how to announce Resource availability restrictions.
