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
  
  ```sh
  ls EXAMPLE/configs
  # should output something like:
  # file1.cfg file2.cfg file3.cfg ...
  ```

  2. Start the `batfish` Docker service.

  ``` sh
  docker run --name batfish -v batfish-data:/data -p 8888:8888 -p 9997:9997 -p 9996:9996 batfish/allinone
  # after the first run, you can start it again using "docker start batfish" or "docker restart batfish"
  ```

  3. Extract the JSON representation of the Batfish AST using angler.
  
  ``` sh
  cd angler
  poetry install
  poetry run python -m angler --full-run --simplify-bools EXAMPLE
  # you should obtain an [EXAMPLE.json] Batfish JSON file 
  # and an [EXAMPLE.angler.json] Angler JSON file as output
  # you can also perform these two steps separately by running
  # poetry run python -m angler EXAMPLE
  # followed by
  # poetry run python -m angler --simplify-bools EXAMPLE.json
  ```
  
  4. Verify your desired property of the network using [Timepiece](https://github.com/NetworkVerification/Timepiece).
  
  ``` sh
  cd Timepiece
  dotnet build Timepiece.Angler
  dotnet run --project Timepiece.Angler -- run EXAMPLE.angler.json [query]
  ```

## Contributing FAQ/Roadmap

  * _How do I make changes to how Angler represents the Batfish AST?_  
  Modify the Batfish AST files in [angler](https://github.com/NetworkVerification/angler).
  See <https://github.com/NetworkVerification/angler/tree/main/angler/bast>.
  * _How do I make changes to Angler's AST, or how Angler simplifies the Batfish AST to the Angler AST?_  
  Modify the Angler AST files or the conversion code in [angler](https://github.com/NetworkVerification/angler).
  See <https://github.com/NetworkVerification/angler/tree/main/angler/aast> and
  <https://github.com/NetworkVerification/angler/tree/main/angler/convert.py>.
  * _How do I make changes to how Timepiece models the Angler AST?_  
  Modify the 
  [Timepiece.Angler AST](https://github.com/NetworkVerification/Timepiece/tree/main/Timepiece.Angler/Ast) 
  or the [AST-to-Zen encoding](https://github.com/NetworkVerification/Timepiece/tree/main/Timepiece.Angler/Ast/AstState.cs)
  * _How do I make changes to what properties Timepiece can check on an Angler file?_  
  Add a new [QueryType](https://github.com/NetworkVerification/Timepiece/tree/main/Timepiece.Angler/Networks/QueryType.cs)
  for your property, then add code to [Program.cs](https://github.com/NetworkVerification/Timepiece/tree/main/Timepiece.Angler/Program.cs) to
  define how to construct an [AnnotatedNetwork](https://github.com/NetworkVerification/Timepiece/tree/main/Timepiece/Networks/AnnotatedNetwork.cs) that checks the desired property.
