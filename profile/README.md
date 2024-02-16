# Princeton Network Verification

This organization maintains and organizes the [Princeton Programming Languages group](https://pl.cs.princeton.edu)'s
network verification tools and systems.

These tools and systems include:
  * [nv](https://github.com/NetworkVerification/nv), a network control plane intermediate representation language and analysis tool
  * [angler](https://github.com/NetworkVerification/angler), a tool for exporting [Batfish](https://github.com/batfish/batfish) representations
  of network configuration files, for use by other systems
  * [Timepiece](https://github.com/NetworkVerification/Timepiece), a modular network control plane verification tool

## End-to-End Verification Workflow for Timepiece

Install prerequisites: Docker, python and python-poetry, and dotnet.

  1. Acquire your network's configuration files.
  The [angler](https://github.com/NetworkVerification/angler) repository contains some examples.
  We'll imagine we have a directory "`EXAMPLE`" containing a subdirectory "`configs`" with these files.
  2. Start the `batfish` Docker service.

  ``` sh
  docker run --name batfish -v batfish-data:/data -p 8888:8888 -p 9997:9997 -p 9996:9996 batfish/allinone
  # after the first run, you can start it again using "docker start batfish"
  ```

  3. Extract the JSON representation of the Batfish AST using angler.
  
  ``` sh
  cd angler
  poetry install
  poetry run python -m angler --full-run --simplify-bools EXAMPLE
  # you should obtain an [EXAMPLE.json] Batfish JSON file 
  # and an [EXAMPLE.angler.json] Angler JSON file as output
  ```
  
  4. Verify your desired property of the network using [Timepiece](https://github.com/NetworkVerification/Timepiece).
  
  ``` sh
  dotnet build Timepiece.Angler
  dotnet run --project Timepiece.Angler -- run EXAMPLE.angler.json [query]
  ```

