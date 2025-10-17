# Knowledge Base

## Convertigo server access
- Docker remains unavailable in this Codex Cloud workspace because the kernel lacks the permissions required by `iptables`, so `dockerd` exits immediately even after installing `docker.io`.
- As a workaround we can run Convertigo 8.3.9 on Tomcat 9 with Java 21:
  1. Work outside the Git repo (e.g. `/workspace/tmp`).
  2. Download Tomcat 9.0.89 and the `convertigo-8.3.9.war` release.
  3. Extract Tomcat, create `webapps/convertigo`, and unzip the WAR there.
  4. Start the server with `bin/catalina.sh start` and stop it with `bin/catalina.sh stop`.
  5. On first start Tomcat creates `/root/convertigo`; symlink the repo with `ln -s /workspace/c8oprj-c8o-mcp /root/convertigo/projects/ConvertigoMCP`.
- The Convertigo console is reachable at `http://localhost:8080/convertigo` once Tomcat is running.
- Prime the engine by calling `/convertigo/projects/ConvertigoMCP/.json?__sequence=EXEC&script=<js>` so the project is loaded into the workspace. Once primed, `/convertigo/api/exec` succeeds again and returns the Rhino output (e.g. `{"output": "8.3.9"}` for `return com.twinsoft.convertigo.engine.ProductVersion.productVersion`).

## Useful commands
```sh
# Start Tomcat after unpacking the WAR under apache-tomcat-9.0.89/webapps/convertigo
bin/catalina.sh start

# Stop Tomcat
bin/catalina.sh stop

# Query the Convertigo engine version through the EXEC sequence (primes the project)
curl "http://localhost:8080/convertigo/projects/ConvertigoMCP/.json?__sequence=EXEC&script=return%20com.twinsoft.convertigo.engine.ProductVersion.productVersion"

# Same script through /convertigo/api/exec once the project is loaded
curl "http://localhost:8080/convertigo/api/exec" \
  -H "Content-Type: text/plain" \
  -d "return com.twinsoft.convertigo.engine.ProductVersion.productVersion"

# Retrieve live engine metrics (SimpleStep + JsonToXmlStep sequence)
curl "http://localhost:8080/convertigo/projects/ConvertigoMCP/.json?__sequence=EngineMetrics"
curl "http://localhost:8080/convertigo/api/metrics"
```
