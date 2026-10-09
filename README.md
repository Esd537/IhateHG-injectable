IhateHG injetavel graças ao bypassed https://codeberg.org/angelwhisp/ihatehg-injectable : 
# ihatehg-injectable

An independently published fork of [/antimousegamer/ihatehg](https://github.com/antimousegamer/ihatehg), extended with ClassTransform, JVMTI retransformation, and a single-file native loader. This repository preserves the original Java client source alongside the injector source.

A single-file Windows x64 loader for the IHateHG Minecraft 1.8.9 client, powered by [ClassTransform](https://github.com/Lenni0451/ClassTransform), JNI, and JVMTI.

## Requirements

- Windows x64 and an x64 Java process running Minecraft 1.8.9.
- A compatible MCP- or SRG-named runtime with JVMTI retransformation support.

Users do not need a JDK, Attach API, a separate DLL, or a separate JAR. Launcher detection does not guarantee compatibility with every launcher or version.

## How to use

1. Download `loader.exe` from the release assets when available.
2. Start Minecraft 1.8.9 and wait for the game to finish loading.
3. Run `loader.exe`.
4. The loader detects `java.exe` and `javaw.exe`, prioritizing Minecraft, Lunar, and Badlion windows. If multiple candidates remain, select the intended process from the numbered list.
5. Wait for the success message. The console closes after three seconds.
6. Press **Insert** in the game to open the client GUI.

```text
https://discord.gg/ANfdGTW99B
[+] Transforming classes...
[+] Hooked
[+] Done, sucessfully injected
[/] Closing in 3s...
```

Console title: `made by rogerio, forked by angel`.

### Command-line options

```bat
loader.exe --list
loader.exe 12345
loader.exe 12345 --rollback
```

Replace `12345` with the game process ID. Rollback restores transformed method bodies; it does not unload the agent or reset Java objects and static fields.

The embedded DLL is extracted automatically to `%LOCALAPPDATA%\IHateHG\<DLL SHA256>\ihatehg_injector.dll`. The selected JAR is extracted to the user's temporary directory and scheduled for deletion when the JVM exits. `ihatehg-native.log` is written beside the cached DLL. Keep the cache while the game is running.

### Client commands

The default command prefix is `-`:

```text
-config save example
-config load example
-prefix !
-setvalue <setting> <value>
```

## How to build

### Tools

- JDK 8 for the legacy ForgeGradle Java build.
- Visual Studio 2022 with Desktop development with C++ and the Windows SDK.
- CMake 3.20 or newer.
- JNI headers from a JDK; set `JAVA_HOME` if CMake cannot find them.
- Internet access for the Gradle wrapper and dependencies.

Open an **x64 Native Tools Command Prompt for VS 2022** in the repository root. Set `JAVA_HOME` to your JDK 8 installation:

```bat
set "JAVA_HOME=C:\path\to\jdk8"
set "PATH=%JAVA_HOME%\bin;%PATH%"
gradlew.bat --no-daemon injectorMcpJar reobfInjectorJar
cmake -S injector -B injector/build -A x64 -DPAYLOAD_JAR="%CD%/build/libs/ihatehg-2.1-injector.jar" -DPAYLOAD_MCP_JAR="%CD%/build/libs/ihatehg-2.1-injector-mcp.jar"
cmake --build injector/build --config Release
copy /Y injector\build\dist\IHateHGInjector.exe injector\build\dist\loader.exe
```

Distribute **only `injector/build/dist/loader.exe`**. The DLL in that directory is an intermediate artifact already embedded in the EXE. The native runtime is statically linked.

Do not publish your working directory, build caches, logs, reports, or local toolchains.

## Architecture

- `src/main/java`: client, ClassTransform manager, bridge, and game hooks.
- `src/main/resources/mappings/1.8.9`: MCP/SRG mapping tables.
- `injector/agent/src`: embedded JARs, JNI bootstrap, class-loader integration, and JVMTI callbacks.
- `injector/console/src`: console, process detection, embedded DLL extraction, and native loading.
- `injector/tests`: Java unit tests and isolated integration fixtures.
- `injector/bridge/src`: optional Attach bridge, not included in the loader build.

The agent selects MCP or SRG bytecode before loading the client. Register transformers in `DefaultTransformers.register`. `TransformManager.init()` enables them and requests target retransformation. JVMTI callbacks delegate bytecode changes to ClassTransform. Retransformation changes method bodies, not class structure.

## Troubleshooting

- **No Java process found:** launch the game first and ensure its JVM is x64.
- **Could not open target process:** use compatible game/loader permissions and note the Win32 error code.
- **JVM bootstrap failed:** inspect the cached `ihatehg-native.log`. Review logs before sharing because they may contain local paths.
- **Mapping or class-loader errors:** another launcher version may require additional mappings or loader support. Restart a crashing session instead of injecting repeatedly.

Standalone-loader tests passed against isolated MCP and SRG fixtures, including detection, transformation, Minecraft getter resolution, rollback, and reactivation. This does not prove universal launcher compatibility.

## Credits and license

Forked from [maththekid/ihatehg](https://github.com/maththekid/ihatehg). This is an independent fork, not an official upstream release. Uses Lenni0451's ClassTransform and ASM. The 1.8.9 mapping tables were imported from the Vape reference supplied during development. Third-party redistribution terms apply separately; preserve upstream notices.

Project license: [MIT](LICENSE).
