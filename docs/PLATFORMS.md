# Platform Support

This project uses a layered platform model:

1. **Core knowledge layer**: `skills/`, `RULES.md`, `routing.md`, and `CTF-Sandbox-Orchestrator/`. This layer is platform-agnostic.
2. **Execution layer**: scripts, local tools, MCP servers, and path conventions. This layer is platform-specific.
3. **Client layer**: Claude Code, Codex CLI, Cursor, Cline, Windsurf, Kiro, or any Agent client that can load rules and call tools.

## Support matrix


## What is shared across platforms

The following are not tied to a platform:

- `RULES.md`
- `skills/SKILL.md`
- `skills/routing.md`
- most `SKILL.md` methodology files
- `CTF-Sandbox-Orchestrator/`
- MCP concepts and JSON configuration shape
- `burp-mcp-full/mcp-bridge.js`
- Java source under `burp-mcp-full/`

## What is platform-specific

The following must be adapted per OS:

- tool installation commands;
- executable names and paths;
- script language (`.ps1` vs `.sh`);
- package managers (`winget`, `apt`, `brew`, `pipx`, `npm`);
- desktop app locations such as IDA Pro and BurpSuite;
- Android SDK / platform-tools paths;
- MCP config file locations for each Agent client.

## Tool coverage by platform


## Recommended routing for setup docs

- Windows users: start from `README.md`.
- Kali users: start from `kali/README-kali.md`.
- Ubuntu / Debian users: start from `docs/platforms/linux.md`.
- macOS users: start from `docs/platforms/macos.md`.
- Kali users should use `kali/scripts/bootstrap-reverse.sh` and `kali/scripts/refresh-tool-index.sh`. Generic Linux/macOS users should use `skills/scripts/bootstrap-reverse.sh` and `skills/scripts/refresh-tool-index.sh`.

## Linux/macOS bootstrap and tool index

For generic Linux/macOS, list supported bootstrap capabilities:

```bash
bash skills/scripts/bootstrap-reverse.sh --list
```

Install or configure capabilities with the same capability names as the Windows PowerShell version:

```bash
bash skills/scripts/bootstrap-reverse.sh jadx apktool frida
bash skills/scripts/bootstrap-reverse.sh jshookmcp anything-analyzer
bash skills/scripts/bootstrap-reverse.sh idapro --start-services
```

Refresh the local tool index only:

```bash
bash skills/scripts/refresh-tool-index.sh
```

This writes:

```text
skills/tool-index.md
skills/tool-index.json
```

The generic Bash bootstrap uses the same core capability names as the Windows PowerShell version. Kali has a dedicated Bash bootstrap that covers those core names plus Kali-native extras. Refresh scripts only detect tools and suggest platform-appropriate installation commands. Use the platform guides for setup and manual-only tools.
