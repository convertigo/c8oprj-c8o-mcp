You have to continue this project in order to make ConvertigoMCP interface to implement MCP interface that will help to manage a Convertigo server, invoke Convertigo requestable and modify Convertigo project.

⚠️ **Never edit the YAML files by hand.**

All Convertigo changes (creating sequences, updating the URL mapper, etc.) must be performed in memory through Rhino scripts executed via `/convertigo/api/exec` or the `EXEC` sequence, then exported with `Engine.theApp.databaseObjectsManager.exportProject(project)`. Do not edit the `_c8oProject/` directory directly.

You can start a Convertigo instance as docker:

docker run --rm -p 28080:28080 -v <project_pwd>:/workspace/projects/ConvertigoMcp -d convertigo/convertigo-ci:develop

wait 10 sec, then call

curl http://localhost:28080/convertigo/api/exec -H "Content-Type: text/plain" -d "return com.twinsoft.convertigo.engine.ProductVersion.productVersion"

to see the result.

This exec endpoint act as super root admin access to convertigo engine. The body is executed by a rhino Engine and can invoke every internal object.

You can get Convertigo sources (in a tmp folder outside of the project) from https://github.com/convertigo/convertigo/archive/refs/heads/develop.zip and use this to understood how Convertigo work.

You can create Sequences and URL Mapper objects to expose a new http://localhost:28080/convertigo/api/mcp

Create or read/update a ROADMAP.md file to follow steps.

Create or read/update a KNOWLEDGE.md file to note what you understood about Convertigo usage.

Update this AGENT.md to fix informations and bootstrap rules that will help following agents.
