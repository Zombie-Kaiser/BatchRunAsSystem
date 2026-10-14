# BatchRunAsSystem

Impersonate **NT AUTHORITY\SYSTEM** and execute commands or launch interactive processes using only native Windows features (Task Scheduler, certutil, PowerShell) — no third-party tools.

## Files

| File | Description |
|---|---|
| `BatchRunAsSystem.bat` | Main script (single self-contained file) |
| `LICENSE` | Apache License 2.0 |

## Requirements

- Windows 10/11 (PowerShell 5.1 built in)
- **An elevated terminal (Run as Administrator) is required.**

## Usage

```
BatchRunAsSystem.bat command [-Application app] [-Argument args]
BatchRunAsSystem.bat process [-CommandLine cmd] [-Hide] [-RedirectKind None/Bind/Reverse] [-Address ip] [-Port port]
BatchRunAsSystem.bat shell [commandline]
BatchRunAsSystem.bat impersonate
BatchRunAsSystem.bat status
```

### command

Run a non-interactive command as SYSTEM via Task Scheduler (register -> run -> unregister, like the original).

```bat
BatchRunAsSystem.bat command
```
Default: `powershell.exe -Command "whoami | Out-File C:\result.txt"` — then check `type C:\result.txt` for `nt authority\system`.

```bat
BatchRunAsSystem.bat command -Application cmd.exe -Argument "/c whoami > C:\result.txt"
```

### process

Launch an interactive SYSTEM process on the active desktop (visible in a new console window).

```bat
BatchRunAsSystem.bat process
BatchRunAsSystem.bat process -CommandLine cmd.exe
BatchRunAsSystem.bat process -Hide
```

For SSH/WinRM sessions without desktop access, redirect the process stdin/stdout/stderr to a socket:

```bat
rem Bind: script waits for one connection, then connect with: nc 127.0.0.1 4444
BatchRunAsSystem.bat process -RedirectKind Bind -Address 0.0.0.0 -Port 4444

rem Reverse: start a netcat listener first (nc -l 4444), then:
BatchRunAsSystem.bat process -RedirectKind Reverse -Address 127.0.0.1 -Port 4444
```

### shell

Shortcut for an interactive SYSTEM console:

```bat
BatchRunAsSystem.bat shell
BatchRunAsSystem.bat shell powershell.exe
```

### impersonate

Token impersonation (the PowerShell module's `Invoke-ImpersonateSystem`) cannot work from a batch context, so this opens an interactive SYSTEM console instead. Exit it to "revert".

```bat
BatchRunAsSystem.bat impersonate
```

### status

Show current user and elevation state.

```bat
BatchRunAsSystem.bat status
```

## Notes

- The `process` action writes a payload to `%SystemRoot%\Temp\BatchRunAsSystem_payload.ps1` and a config file; the payload self-deletes after use, stale files are cleaned on the next run.
- Each action is a separate process: token impersonation state does not carry over between calls (this is why `impersonate` opens a console instead).
- cmd.exe quoting has limits: argument values containing both double-quotes and an ampersand may need manual adjustments.

## Disclaimer

This software is provided "as is", without warranty of any kind. Use only on machines you own or are authorized to administer. See `LICENSE`.
