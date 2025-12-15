You are helping me with a research and documentation task.

Current state:
- Java APIs directly query DB to get PDM
- Payloads are built from PDM and sent to downstream systems

Target state:
- APIs will call a new microservice
- Microservice handles DB interactions
- APIs build payloads from microservice response

My responsibility:
- Research and document EPRO APIs and Vertex Index APIs

Goal:
- Identify data flow across DB → PDM → Java → downstream payload
- Prepare documentation for future microservice design

Rules:
- Do not assume fields exist unless verified in code or DB mappings
- Point to exact Java classes, methods, SQL queries, and workflows
- If information is missing, clearly say "Not Found"
- Organize output in tables
