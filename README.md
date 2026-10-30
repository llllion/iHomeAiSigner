# iHomeAiSigner
代码签名（git-ai）Extension for Trae & Qoder
A Trae & Qoder extension for AI code authorship tracking（代码签名）.

What happens on install
On first activation the extension:

(Default) Uninstalls any existing local git-ai: uninstall-hooks + delete ~/.git-ai
Copies the VSIX-bundled CLI for the current OS/arch into ~/.git-ai/bin/
Creates git / git-og shims and writes git_path when needed
Runs git-ai install to register hooks and set git.path
Windows: starts in user mode, then offers an explicit system-proxy option. Accepting it opens UAC once, installs an ACL-protected proxy under Program Files\\git-ai, updates Machine PATH, and migrates detected IDEs/terminals. Declining keeps the fully functional user-scoped integration.
No network download of the upstream CLI is required for normal product installs.

Uninstall
Preferred: command 代码签名: 完全卸载并清理 — restores Trae / IDEA / VS Code / SourceTree configs, deletes ~/.git-ai, cleans environment traces, then uninstalls the extension.

Environment cleanup is driven by resources/cleanup-env.ps1:

User-scope cleanup runs normally without elevation.
Legacy Machine-scope cleanup only runs after explicit confirmation. Its script is compressed into an in-memory EncodedCommand; no elevated .ps1 is created or executed from %TEMP%.
When an ownership-marked system proxy exists, uninstall requests UAC so Program Files\\git-ai, its ProgramData marker, and Machine PATH can be removed. Declining UAC leaves an explicit partial-uninstall state.
Scope	Cleaned	Needs Admin
User	User Path entries, User BASH_ENV, git-ai blocks in ~/.bashrc / .bash_profile / .profile	No
Machine	Machine Path entries, %ProgramFiles%\Git\etc\profile.d\git-ai-path.sh	Yes (UAC)
The User pass also reports Machine-level leftovers from older releases. Use 清理旧版系统 PATH to remove only those entries, or 完全卸载并清理 during uninstall; both ask for UAC only when leftovers exist.

If you only click the Extensions panel Uninstall button: the hook immediately stages and starts an independent cleanup job, then clears settings/hooks, deletes ~/.git-ai, and runs uninstall-hooks. An owned system proxy triggers UAC; if authorization is declined, the pending marker and system residual report remain instead of reporting complete success. Log: ~/git-ai-extension-uninstall.log. Prefer 完全卸载并清理 so the result can be shown before the extension is removed.

Bundle CLI binaries before packaging
Put platform builds in gi-ai/dist/, then npm run package copies them into cli-binaries/:

npm run package:dev creates a development VSIX that may use an unsigned system helper for local testing.
npm run package creates a production VSIX and refuses to package unless the Windows system helper has a valid Authenticode signature. Run production packaging on Windows after signing the helper; signing must happen before the component manifest is generated.
Windows VSIX build order (required)
Prerequisite: Visual Studio Build Tools with Desktop development with C++ and a Windows SDK. The build script automatically loads the latest installed Build Tools developer environment when link.exe is not already available.

Always build Windows packages from the repository root with:

.\scripts\build-vsix-windows.ps1 -Flavor development

For a signed release, use -Flavor production after configuring Authenticode signing. The script enforces this order:

Build the final git-ai.exe
→ copy it to dist
→ build git-ai-system-helper.exe so it embeds that final payload
→ compare the embedded payload SHA-256 with dist/git-ai-windows-x64.exe
→ package the VSIX

Do not build all Rust binaries in one command and then copy them to dist: Cargo may build the helper before the final payload is copied. copy-static.js independently repeats the embedded-payload hash check, so an inconsistent VSIX cannot be packaged even if this script is bypassed.

File in dist/	Platform
git-ai-windows-x64.exe	Windows x64
git-ai-windows-arm64.exe	Windows ARM64
git-ai-macos-x64	macOS Intel
git-ai-macos-arm64	macOS Apple Silicon
git-ai-linux-x64	Linux x64
git-ai-linux-arm64	Linux ARM64
Customers only need the matching binary for their machine, but one VSIX can contain all of them.

Extension id / Open VSX namespace
publisher in package.json must match your Open VSX namespace: iHomeCoder
Extension id: iHomeCoder.codesign
Customers do not create a namespace. Only the publisher account (your company) registers iHomeCoder on Open VSX once, then publishes. Customers install from marketplace or from the .vsix file.
Install
Search for 代码签名 / id iHomeCoder.codesign, or:

code --install-extension iHomeCoder.codesign-1.0.2.vsix

After install, reload if prompted so git.path picks up the shim.

Commands
Command	Description
代码签名: Check Status	Show CLI path, integration mode, elevation state, bundled binary, setup state
代码签名: 启用系统级 Git 代理	Request UAC and converge Machine PATH, IDEs and terminals on the protected system proxy
代码签名: 切换为仅当前用户模式	Remove the owned system proxy with UAC, then restore user-scoped integration
代码签名: Repair Integration	Re-run shim setup + git-ai install
代码签名: Reinstall CLI	Re-install CLI from the VSIX bundle, then run install
代码签名: Toggle Show AI Code	Toggle blame decorations
代码签名: 完全卸载并清理	还原 Trae/SourceTree/git.path、删 ~/.git-ai，并尝试自动卸载本扩展
Note: Clicking Uninstall in the Extensions panel cannot run cleanup immediately (VS Code/Cursor platform limit). Cleanup runs via vscode:uninstall only after you fully quit the IDE (not Reload). Prefer 完全卸载并清理.

Settings
Setting	Default	Description
gitai.autoInstallCli	true	First setup / upgrade: install or refresh bundled CLI (skipped on steady-state restarts)
gitai.autoRunInstall	true	First setup / upgrade: run git-ai install-hooks (skipped on steady-state restarts)
gitai.replaceLocalInstallOnSetup	true	First setup / Repair: remove local ~/.git-ai then reinstall
gitai.blameMode	line	Gutter blame display mode
License
MIT
