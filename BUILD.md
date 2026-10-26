# Building Slimey for Windows

Slimey can be distributed as a portable Windows application or as a guided installer. Both options bundle the Python runtime and application dependencies, so end users don't need to set up Python themselves.

## Distribution options

| Package | Approx. size | Installation | Recommended use |
|---|---:|---|---|
| Portable build | 400–600 MB | Extract or copy the folder and launch the executable | Development testing and portable use |
| Windows installer | 200–400 MB | Run the installer and follow the setup wizard | Sharing Slimey with other Windows users |

Package sizes vary with dependencies, model libraries, and build configuration.

## 1. Create a portable build

Open a terminal in the project directory and execute:

```bat
build.bat
```

The build script uses PyInstaller to package the application and its dependencies. The first build may take several minutes.

Once complete, the distributable directory is expected at:

```text
dist\Slimey\Slimey.exe
```

Keep the entire `Slimey` directory together when distributing the portable build. The executable relies on bundled resources and libraries inside that directory.

### Running Slimey on another PC

1. Copy the complete distribution folder to the target computer.
2. If required, create a `.env` file alongside the executable using `.env.example` as a reference.
3. Configure the API credentials or local AI services you intend to use.
4. Launch `Slimey.exe`.
5. Look for the application icon in the Windows notification area.

Python does not need to be installed separately when the application has been packaged correctly.

## 2. Build a Windows installer

The installer packages the portable application into a conventional Windows installation experience.

### Prerequisites

Install [Inno Setup 6](https://jrsoftware.org/isdl.php) before building the installer.

### Build command

Run the following from the project root:

```bat
build.bat installer
```

The expected output is:

```text
dist\Setup-Slimey.exe
```

### Installation experience

The user launches the installer and follows its prompts to:

- Select an installation directory.
- Choose whether to create a desktop shortcut.
- Configure startup behavior, if supported by the installer.
- Complete installation and launch Slimey.

The installed application can subsequently be removed through Windows Settings, provided the installer defines the appropriate uninstall configuration.

## 3. What the package includes

| Component | Included | Purpose |
|---|---|---|
| Python runtime | Yes | Executes the application without a separate Python installation |
| PyQt6 | Yes | Desktop interface and UI components |
| faster-whisper and CTranslate2 | If collected by the build | Local speech recognition |
| edge-tts | If collected by the build | Text-to-speech output |
| LLM provider SDKs | If collected by the build | Connects to supported AI providers |
| Environment secrets | No | Users supply their own credentials |
| Ollama server and models | No | Installed and managed separately when using local inference |

The exact package contents depend on the build configuration and the dependencies included in the project.

**Never bundle personal API keys or a populated `.env` file in a distributable release.** Include only a template containing empty values or safe placeholders.

## 4. First launch checklist

Use this checklist when running Slimey on a new Windows machine.

### Windows SmartScreen

Windows may display a warning for an unsigned executable.

- Check that the application came from a trusted source.
- Use the available Windows options only when you trust the build.
- For public distribution, consider signing the executable and installer with an appropriate code-signing certificate.

Signing can improve publisher verification, but it does not guarantee that every security warning will disappear.

### Microphone access

If voice input is enabled, grant microphone permission when Windows requests it. Verify the selected input device in Windows sound settings if speech recognition does not work.

### Notification-area icon

Check the notification area near the Windows clock for Slimey's icon. If it is missing, inspect Task Manager for the application process and check the application logs for startup errors.

### Verify AI and voice functionality

1. Start Slimey and invoke its configured voice shortcut.
2. Ask a basic question that exercises the assistant's response pipeline.
3. Test screen-aware functionality separately if the required vision model is configured.
4. If the assistant cannot respond, check provider credentials, local model availability, and service connectivity.

For local inference, make sure Ollama is running and that the required model has been downloaded. For hosted services, verify the relevant API credentials and network access.

## 5. Troubleshooting build failures

### PyInstaller is unavailable

If the build reports that PyInstaller cannot be found, install it in the active build environment:

```bat
python -m pip install pyinstaller
```

Retry the build after confirming that the terminal is using the intended Python environment.

### faster-whisper collection fails

Upgrade the relevant packages in the build environment:

```bat
python -m pip install --upgrade faster-whisper ctranslate2
```

If collection continues to fail, inspect the complete build log for missing modules, incompatible versions, or unsupported binaries.

### DLL loading errors on another computer

A packaged application can still depend on operating-system components or native runtime libraries. If a DLL error occurs, identify the missing dependency from the error message before installing additional software.

Some Windows systems may require the appropriate [Microsoft Visual C++ Redistributable](https://aka.ms/vs/17/release/vc_redist.x64.exe).

### Antivirus flags the executable

Packaging tools and self-extracting applications can occasionally trigger security detections. Verify the artifact's origin and investigate the detection rather than assuming it is harmless.

For releases intended for wider distribution, use reproducible build practices and consider code signing.

### The distribution is too large

Large machine-learning and numerical-computing dependencies can significantly increase package size.

To reduce the output:

1. Inspect the dependency tree to identify large packages.
2. Determine whether local speech recognition is required.
3. Exclude unnecessary dependencies only after checking that the application no longer imports them.
4. Rebuild and test every affected feature.

Removing faster-whisper, for example, would require an alternative speech-recognition path if local transcription is no longer available.

### Builds are slow

The first PyInstaller build performs analysis and collects dependencies. Subsequent builds may be faster because intermediate build artifacts can be reused.

Avoid deleting the build cache unnecessarily during routine development. If stale artifacts cause inconsistent results, clean the build output and perform a fresh build.

## 6. Optional: Sign the release

Code signing can establish publisher identity and help users verify that an executable has not been altered after signing.

A typical Windows release workflow uses Microsoft's SignTool and a valid code-signing certificate.

After configuring the signing environment, sign the final artifacts using the appropriate certificate and timestamping service. Keep private keys and certificate passwords out of source control and build logs.

Consult the certificate provider's instructions and Microsoft's SignTool documentation for the correct signing options.

## 7. Expected output structure

After a successful build, the distribution directory should resemble the following:

```text
Slimey/
├── build/                       # Intermediate build artifacts
└── dist/
    ├── Slimey/                  # Portable distribution
    │   ├── Slimey.exe
    │   ├── _internal/           # Bundled runtime and dependencies
    │   ├── .env.example
    │   ├── LICENSE
    │   └── README.md
    └── Setup-Slimey.exe         # Installer, when requested
```

The exact layout depends on the PyInstaller configuration. Some configurations place dependencies beside the executable rather than inside an `_internal` directory.

Before distributing a release, test the packaged application on a clean Windows environment and verify startup, microphone access, AI connectivity, and any features that depend on external services.
