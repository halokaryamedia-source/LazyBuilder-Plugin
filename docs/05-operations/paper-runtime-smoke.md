# Paper Runtime Smoke Verification

This is the minimum live-server proof for LazyBuilder Paper plugins. It complements `mvn verify`; it does not replace unit tests or the full world-lifecycle validation checklist.

## Goal

Prove that the packaged Paper server can actually boot with the current `World-Manager` and `Utilities-Manager` JARs, that both plugins reach their enabled state, and that the server can be stopped cleanly.

This catches a class of failures that source/unit verification cannot prove, including plugin metadata errors, runtime linkage failures, Paper API incompatibilities, startup lifecycle failures, and packaged-JAR mistakes.

## Prerequisites

- Java 21 available on `PATH`, or pass `-JavaExecutable`.
- A known Paper 1.21.4 server JAR already available locally.
- Current Paper modules built from the repository root:

```powershell
mvn verify
```

## Run

From the repository root:

```powershell
powershell -ExecutionPolicy Bypass -File scripts/verify-paper-runtime.ps1 `
  -ServerJar "E:\1.21.4\paper.jar"
```

If the Paper JAR has another filename, provide that exact path.

The harness uses the verified module artifacts by default:

```text
plugins/world-manager/target/World-Manager-0.1.0-SNAPSHOT.jar
plugins/utilities-manager/target/Utilities-Manager-0.1.0-SNAPSHOT.jar
```

It creates an isolated disposable server under:

```text
.runtime-proof/paper-smoke/
```

The existing live server is not modified.

## Pass conditions

The smoke proof passes only when:

1. Paper reaches its normal ready state.
2. `World-Manager enabled.` appears in runtime output.
3. `Utilities-Manager enabled` appears in runtime output.
4. No known fatal plugin startup signal is detected.
5. The disposable Paper process accepts a normal `stop` command; forced termination is used only as cleanup after a timeout.

## What this does not prove

A passing smoke run does not prove world operations. Stable-release validation still requires a disposable live-server scenario covering:

```text
create
-> teleport
-> settings
-> unload/load
-> duplicate
-> backup
-> export
-> import
-> delete
-> restart
-> recovery / registry-filesystem consistency
```

Large-file transfer throughput and at least one real conversion workflow remain separate manual/live proof items.

## Failure handling

Do not weaken the harness merely to make it green. If startup fails, inspect the disposable runtime output and fix the actual plugin/runtime incompatibility.

Do not add a second plugin bootstrap, compatibility manager, dependency injection container, or alternate Paper adapter to solve an isolated runtime failure unless the failure demonstrates a real architectural requirement.
