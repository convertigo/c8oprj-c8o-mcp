You have to continue this project in order to make ConvertigoMCP interface to implement MCP interface that will help to manage a Convertigo server, invoke Convertigo requestable and modify Convertigo project.

⚠️ **Never edit the YAML files by hand.**

All Convertigo changes (creating sequences, updating the URL mapper, etc.) must be performed in memory through Rhino scripts executed via `/convertigo/api/exec` or the `EXEC` sequence, then exported with `Engine.theApp.databaseObjectsManager.exportProject(project)`. Do not edit the `_c8oProject/` directory directly.

You can start a Convertigo instance as docker:

```
docker run --rm -p 28080:28080 -v <project_pwd>:/workspace/projects/ConvertigoMcp -d convertigo/convertigo-ci:develop
```

Wait 10 sec, then call:

```
curl http://localhost:28080/convertigo/projects/ConvertigoMCP/.json?__sequence=EXEC \
  -G --data-urlencode "script=return com.twinsoft.convertigo.engine.ProductVersion.productVersion"
```

The first call forces the engine to load the `ConvertigoMCP` project into the workspace. Once the project is live, the legacy admin endpoint is also available:

```
curl http://localhost:28080/convertigo/api/exec \
  -H "Content-Type: text/plain" \
  -d "return com.twinsoft.convertigo.engine.ProductVersion.productVersion"
```

This `exec` endpoint acts as a super root admin access to Convertigo engine. The body is executed by a Rhino engine and can invoke every internal object.

You can get Convertigo sources (in a tmp folder outside of the project) from https://github.com/convertigo/convertigo/archive/refs/heads/develop.zip and use this to understand how Convertigo works.

You can create Sequences and URL Mapper objects to expose a new `http://localhost:28080/convertigo/api/mcp`.

Create or read/update a `ROADMAP.md` file to follow steps.

Create or read/update a `KNOWLEDGE.md` file to note what you understood about Convertigo usage.

Update this `AGENT.md` to fix informations and bootstrap rules that will help following agents.

## Local Convertigo setup without Docker (2025-10-17)
When Docker is unavailable in Codex Cloud, you can run Convertigo 8.3.9 on Tomcat 9 with Java 21:

1. Work outside of this Git repository (e.g. `/workspace/tmp`).
2. Download and extract Tomcat 9.0.89:
   ```sh
   curl -LO https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.89/bin/apache-tomcat-9.0.89.tar.gz
   tar -xzf apache-tomcat-9.0.89.tar.gz
   ```
3. Download the Convertigo 8.3.9 WAR:
   ```sh
   curl -LO https://github.com/convertigo/convertigo/releases/download/8.3.9/convertigo-8.3.9.war
   ```
4. Unpack the WAR into `webapps/convertigo` inside the Tomcat directory:
   ```sh
   mkdir -p apache-tomcat-9.0.89/webapps/convertigo
   unzip convertigo-8.3.9.war -d apache-tomcat-9.0.89/webapps/convertigo
   ```
5. Start Tomcat in the background (Java 21 is already available in this workspace):
   ```sh
   cd apache-tomcat-9.0.89
   bin/catalina.sh start
   ```
6. On first start Tomcat creates the Convertigo home under `/root/convertigo`. Link this repository so the engine sees the project:
   ```sh
   ln -s /workspace/c8oprj-c8o-mcp /root/convertigo/projects/ConvertigoMCP
   ```
7. Stop the server with `bin/catalina.sh stop` when you are done.

The Convertigo web console is available on `http://localhost:8080/convertigo`. Once the repository is symlinked into `/root/convertigo/projects/ConvertigoMCP`, you may need to prime the engine with a call to `http://localhost:8080/convertigo/projects/ConvertigoMCP/.json?__sequence=EXEC&script=<js>` so the project is loaded. After that warm-up the legacy admin endpoint (`http://localhost:8080/convertigo/api/exec`) responds again. Engine activity is logged under `/root/convertigo/logs/engine.log` if you need to debug requests.

### Hot-updating the Convertigo project
You can extend the project dynamically through Rhino code executed via the `EXEC` sequence or the `/convertigo/api/exec` endpoint:

1. Retrieve the project instance: `var project = Packages.com.twinsoft.convertigo.engine.Engine.theApp.databaseObjectsManager.getOriginalProjectByName("ConvertigoMCP");`
2. Add or modify sequences and URL mapper entries through the Java API exposed to Rhino.
3. When the in-memory changes are ready, persist them to disk with `Packages.com.twinsoft.convertigo.engine.Engine.theApp.databaseObjectsManager.exportProject(project);` so Git can capture the updates.

Convertigo accepts these changes without a server restart. Remember to commit the exported files so the repository stays in sync with the live configuration.

### MCP tooling (2025-10-17)
- `EngineMetrics` is a generic sequence composed of a `SimpleStep` that builds a Rhino object with engine statistics (memory usage, active sessions, worker threads, contexts, request rate, uptime, etc.) and a `JsonToXmlStep` that exposes the data. Call it directly with `http://localhost:8080/convertigo/projects/ConvertigoMCP/.json?__sequence=EngineMetrics`.
- The URL mapper now forwards `GET /convertigo/api/metrics` to the `EngineMetrics` sequence so tools can retrieve the same JSON payload without running arbitrary scripts.
- `ListProjects` exposes the set of Convertigo projects through `GET /convertigo/api/mcp/projects` (or the direct sequence call `...__sequence=ListProjects`).
- `ListProjectSequences` returns the requestables of a given project. Call it with `GET /convertigo/api/mcp/sequences?project=ConvertigoMCP` to obtain names, comments, and class names.
- `DescribeDatabaseObject` surfaces the logical tree of a project or a specific database object. Call `GET /convertigo/api/mcp/tree?depth=2` to expand two levels, or add `qname=<project.qname>` to target a single branch.
- `ListDatabaseObjectCandidates` mirrors the Studio palette to reveal which database object classes can be added under a parent. Invoke it with `GET /convertigo/api/mcp/candidates?qname=<dbo.qname>&folder=<optional-folder-shortname>`.
- `InvokeSequence` wraps the internal requester so MCP clients can execute sequences without `/api/exec`. Use `POST /convertigo/api/mcp/sequences/invoke` with a JSON `payload` string to forward variables.
- `GetDatabaseObjectProperties` (→ `GET /convertigo/api/mcp/properties?qname=<dbo.qname>`) returns property descriptors, types, and current values so the MCP client can render an editor safely.
- `UpdateDatabaseObjectProperties` (→ `POST /convertigo/api/mcp/properties/update`) accepts a JSON object mapping property names to values, validates conversions, applies the setters, and exports the project on success.
- `CreateDatabaseObject` (→ `POST /convertigo/api/mcp/objects`) instantiates a database object class under a parent, optionally applies initial properties, and exports the containing project.
- `ReorderDatabaseObjects` (→ `POST /convertigo/api/mcp/objects/reorder`) reassigns sibling priorities according to the supplied list of qnames and persists the ordering.
- Next targets for the MCP tooling are documented in `ROADMAP.md`: shape the MCP contract (schemas, guardrails) so clients can drive these new endpoints confidently.

## Git workflow tips (2024-10-17)
- Before running `make_pr`, make sure the branch is aligned with the upstream repository. If a previous PR was already merged, run `git fetch origin` followed by `git rebase origin/main` (or the relevant target branch) so the new diff is computed on top of the latest state.
- When Codex Cloud reports merge conflicts during the PR creation step, abort the PR, synchronize the branch with the commands above, resolve any conflicts locally, then rerun `make_pr`.
- If the workspace becomes out of sync with the remote state and you only need the latest files, you can reset the branch with `git fetch origin && git reset --hard origin/main`. This discards local commits, so ensure important work has been exported first.

## Environment notes (2024-10-17)
- The workspace initially lacks the `docker` CLI but `apt-get install docker.io` succeeds. However, starting the daemon fails inside this container: `dockerd` exits with `failed to start daemon: ... iptables v1.8.10 (nf_tables): Could not fetch rule set generation id: Permission denied`. This indicates missing kernel capabilities (e.g., `CAP_NET_ADMIN`) so Docker cannot be used even after installation.
- When Docker is unavailable, use the Tomcat+WAR procedure above to run Convertigo locally on port 8080.
- Coordinate with the requester for an external Convertigo endpoint or another environment where Docker can run, otherwise the MCP interface cannot yet be tested locally.
