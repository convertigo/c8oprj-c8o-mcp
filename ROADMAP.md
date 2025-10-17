# Roadmap

## Stage 1 – Local Convertigo runtime
- [x] Confirm Docker cannot run inside Codex Cloud (daemon fails with missing `iptables` capabilities).
- [x] Download Tomcat 9.0.89 and the Convertigo 8.3.9 WAR outside the repo, unpack the WAR under `webapps/convertigo`, and start Tomcat with Java 21.
- [x] Link this repository into `/root/convertigo/projects/ConvertigoMCP` so the engine sees the project workspace.
- [x] Discover a supported way to execute arbitrary scripts/operations on Convertigo 8.3.9.
  - [x] Inspect bundled admin services and URL mapper definitions to find or expose an equivalent capability (the `EXEC` sequence is mapped at `/convertigo/projects/ConvertigoMCP/.json?__sequence=EXEC&script=<js>`).
  - [x] Confirm that once the project is primed through the sequence endpoint, the legacy `/convertigo/api/exec` endpoint responds again.
- [x] Create a dedicated `EngineMetrics` sequence (SimpleStep + JsonToXmlStep) and map it to `GET /convertigo/api/metrics` so MCP tools can retrieve runtime stats without arbitrary code execution.
- [ ] Update the MCP integration to call that endpoint and retrieve the engine product version as a first test.

## Current status (2025-10-17)
- Tomcat reports a successful startup (`bin/catalina.sh start` → `Tomcat started.`) and creates the Convertigo home under `/root/convertigo`.
- Calling `http://localhost:8080/convertigo/projects/ConvertigoMCP/.json?__sequence=EXEC&script=return%20com.twinsoft.convertigo.engine.ProductVersion.productVersion` returns `{ "output": "8.3.9" }`, priming the project so `/convertigo/api/exec` can also process the same script payload.
- `/convertigo/api/metrics` now returns the `EngineMetrics` payload (memory usage, sessions, worker threads, contexts, request rate, uptime...).
- Next focus: wire the MCP interface against the new metrics endpoint (and eventually other sequences) so the toolset can surface engine health without requiring `exec` access.
