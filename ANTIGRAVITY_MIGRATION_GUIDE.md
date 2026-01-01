# Antigravity Unity Editor Integration - Migration Guide

이 가이드는 Cursor Unity Editor 통합 프로젝트를 기반으로 Antigravity Editor 지원 프로젝트를 생성하는 방법을 설명합니다.

## 📋 목차

1. [개요](#개요)
2. [사전 준비사항](#사전-준비사항)
3. [파일별 수정 가이드](#파일별-수정-가이드)
4. [검증 방법](#검증-방법)
5. [문제 해결](#문제-해결)

---

## 개요

### 프로젝트 정보
- **소스 프로젝트**: com.unity.ide.cursor (v2.0.27)
- **타겟 프로젝트**: com.unity.ide.antigravity
- **기반 에디터**: Antigravity (Windsurf 기반)

### 주요 변경 사항
Cursor의 모든 참조를 Antigravity로 변경하며, 다음 기능들을 유지합니다:
- 자동 설치 탐지
- 워크스페이스 지원
- 기존 창 재사용 옵션
- 프로젝트 파일 생성 (.csproj, .sln)
- Unity 디버거 통합

---

## 사전 준비사항

### 1. Antigravity 설치 정보 확인

작업 전에 다음 정보를 확인해야 합니다:

#### Windows
```powershell
# 설치 경로 확인
Get-ChildItem "C:\Users\$env:USERNAME\AppData\Local\Programs" -Recurse -Filter "*antigravity*"
Get-ChildItem "C:\Program Files" -Recurse -Filter "*antigravity*"

# 프로세스 이름 확인 (Antigravity 실행 후)
Get-Process | Where-Object {$_.ProcessName -like "*antigravity*"}

# 설정 저장 경로 확인
Get-ChildItem "C:\Users\$env:USERNAME\AppData\Roaming" -Directory | Where-Object {$_.Name -like "*antigravity*"}
```

#### macOS
```bash
# 설치 경로 확인
ls -la /Applications/ | grep -i antigravity

# 프로세스 이름 확인 (Antigravity 실행 후)
ps aux | grep -i antigravity

# 설정 저장 경로 확인
ls -la ~/Library/Application\ Support/ | grep -i antigravity
```

#### Linux
```bash
# 설치 경로 확인
which antigravity
ls -la /usr/bin/ | grep antigravity
ls -la /usr/local/bin/ | grep antigravity

# 프로세스 이름 확인 (Antigravity 실행 후)
ps aux | grep -i antigravity

# 설정 저장 경로 확인
ls -la ~/.config/ | grep -i antigravity
```

### 2. 확인해야 할 정보 체크리스트

- [ ] 실행 파일 이름 (예: `antigravity.exe`, `Antigravity.app`)
- [ ] 실행 파일 전체 경로
- [ ] 프로세스 이름 (대소문자 정확히)
- [ ] 설정 저장 디렉토리 이름 (대소문자 정확히)
- [ ] 워크스페이스 설정 파일 경로 (`workspaceStorage` 존재 여부)

---

## 파일별 수정 가이드

### 1. package.json

**파일 위치**: `package.json`

**수정 내용**:
```json
{
  "name": "com.yourname.ide.antigravity",
  "displayName": "Antigravity Editor",
  "description": "Antigravity editor integration for supporting Antigravity as code editor for unity. Adds support for generating csproj files for intellisense purposes, auto discovery of installations, etc.",
  "version": "2.0.27",
  "unity": "2019.4",
  "unityRelease": "25f1",
  "dependencies": {
    "com.unity.test-framework": "1.1.9"
  },
  "_upm": {
    "changelog": "Integration:\n\n- Initial Antigravity support based on Cursor integration"
  }
}
```

**변경 항목**:
- `name`: 패키지 고유 이름 (com.yourname.ide.antigravity)
- `displayName`: Unity 에디터에 표시될 이름
- `description`: 패키지 설명
- `_upm.changelog`: 변경 이력

---

### 2. Editor/VisualStudioCursorInstallation.cs

**파일 위치**: `Editor/VisualStudioCursorInstallation.cs`

이 파일은 가장 많은 수정이 필요한 핵심 파일입니다.

#### 2.1 클래스명 변경 (17줄)

**변경 전**:
```csharp
internal class VisualStudioCursorInstallation : VisualStudioInstallation
```

**변경 후**:
```csharp
internal class VisualStudioAntigravityInstallation : VisualStudioInstallation
```

#### 2.2 상수 변경 (23줄)

**변경 전**:
```csharp
internal const string ReuseExistingWindowKey = "cursor_reuse_existing_window";
```

**변경 후**:
```csharp
internal const string ReuseExistingWindowKey = "antigravity_reuse_existing_window";
```

#### 2.3 실행 파일 탐지 로직 (71-80줄)

**변경 전**:
```csharp
private static bool IsCandidateForDiscovery(string path)
{
#if UNITY_EDITOR_OSX
    return Directory.Exists(path) && Regex.IsMatch(path, ".*Cursor.*.app$", RegexOptions.IgnoreCase);
#elif UNITY_EDITOR_WIN
    return File.Exists(path) && Regex.IsMatch(path, ".*Cursor.*.exe$", RegexOptions.IgnoreCase);
#else
    return File.Exists(path) && path.EndsWith("cursor", StringComparison.OrdinalIgnoreCase);
#endif
}
```

**변경 후**:
```csharp
private static bool IsCandidateForDiscovery(string path)
{
#if UNITY_EDITOR_OSX
    return Directory.Exists(path) && Regex.IsMatch(path, ".*Antigravity.*.app$", RegexOptions.IgnoreCase);
#elif UNITY_EDITOR_WIN
    return File.Exists(path) && Regex.IsMatch(path, ".*Antigravity.*.exe$", RegexOptions.IgnoreCase);
#else
    return File.Exists(path) && path.EndsWith("antigravity", StringComparison.OrdinalIgnoreCase);
#endif
}
```

**주의**: 실제 Antigravity 실행 파일명에 맞게 조정 필요

#### 2.4 설치 인스턴스 생성 (136-142줄)

**변경 전**:
```csharp
installation = new VisualStudioCursorInstallation()
{
    IsPrerelease = isPrerelease,
    Name = "Cursor" + (isPrerelease ? " - Insider" : string.Empty) + (version != null ? $" [{version.ToString(3)}]" : string.Empty),
    Path = editorPath,
    Version = version ?? new Version()
};
```

**변경 후**:
```csharp
installation = new VisualStudioAntigravityInstallation()
{
    IsPrerelease = isPrerelease,
    Name = "Antigravity" + (isPrerelease ? " - Insider" : string.Empty) + (version != null ? $" [{version.ToString(3)}]" : string.Empty),
    Path = editorPath,
    Version = version ?? new Version()
};
```

#### 2.5 설치 경로 탐색 (151-169줄)

**변경 전**:
```csharp
public static IEnumerable<IVisualStudioInstallation> GetVisualStudioInstallations()
{
    var candidates = new List<string>();

#if UNITY_EDITOR_WIN
    var localAppPath = IOPath.Combine(Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData), "Programs");
    var programFiles = IOPath.Combine(Environment.GetFolderPath(Environment.SpecialFolder.ProgramFiles));

    foreach (var basePath in new[] { localAppPath, programFiles }) {
        candidates.Add(IOPath.Combine(basePath, "cursor", "cursor.exe"));
    }
#elif UNITY_EDITOR_OSX
    var appPath = IOPath.Combine(Environment.GetFolderPath(Environment.SpecialFolder.ProgramFiles));
    candidates.AddRange(Directory.EnumerateDirectories(appPath, "Cursor*.app"));
#elif UNITY_EDITOR_LINUX
    candidates.Add("/usr/bin/cursor");
    candidates.Add("/bin/cursor");
    candidates.Add("/usr/local/bin/cursor");
    candidates.AddRange(GetXdgCandidates());
#endif

    foreach (var candidate in candidates.Distinct())
    {
        if (TryDiscoverInstallation(candidate, out var installation))
            yield return installation;
    }
}
```

**변경 후**:
```csharp
public static IEnumerable<IVisualStudioInstallation> GetVisualStudioInstallations()
{
    var candidates = new List<string>();

#if UNITY_EDITOR_WIN
    var localAppPath = IOPath.Combine(Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData), "Programs");
    var programFiles = IOPath.Combine(Environment.GetFolderPath(Environment.SpecialFolder.ProgramFiles));

    foreach (var basePath in new[] { localAppPath, programFiles }) {
        candidates.Add(IOPath.Combine(basePath, "antigravity", "antigravity.exe"));
    }
#elif UNITY_EDITOR_OSX
    var appPath = IOPath.Combine(Environment.GetFolderPath(Environment.SpecialFolder.ProgramFiles));
    candidates.AddRange(Directory.EnumerateDirectories(appPath, "Antigravity*.app"));
#elif UNITY_EDITOR_LINUX
    candidates.Add("/usr/bin/antigravity");
    candidates.Add("/bin/antigravity");
    candidates.Add("/usr/local/bin/antigravity");
    candidates.AddRange(GetXdgCandidates());
#endif

    foreach (var candidate in candidates.Distinct())
    {
        if (TryDiscoverInstallation(candidate, out var installation))
            yield return installation;
    }
}
```

**주의**: 사전 확인한 실제 설치 경로로 수정 필요

#### 2.6 프로세스 탐지 (503-567줄)

**변경 전**:
```csharp
private Process FindRunningCursorWithSolution(string solutionPath)
{
    // ... 경로 정규화 코드 ...

    var processes = new List<Process>();

#if UNITY_EDITOR_OSX
    processes.AddRange(Process.GetProcessesByName("Cursor"));
    processes.AddRange(Process.GetProcessesByName("Cursor Helper"));
#elif UNITY_EDITOR_LINUX
    processes.AddRange(Process.GetProcessesByName("cursor"));
    processes.AddRange(Process.GetProcessesByName("Cursor"));
#else
    processes.AddRange(Process.GetProcessesByName("cursor"));
#endif

    foreach (var process in processes)
    {
        try
        {
            var workspaces = ProcessRunner.GetProcessWorkspaces(process);
            // ... 워크스페이스 비교 로직 ...
        }
        catch (Exception ex)
        {
            Debug.LogError($"[Cursor] Error checking process: {ex}");
            continue;
        }
    }
    return null;
}
```

**변경 후**:
```csharp
private Process FindRunningAntigravityWithSolution(string solutionPath)
{
    // ... 경로 정규화 코드 (동일) ...

    var processes = new List<Process>();

#if UNITY_EDITOR_OSX
    processes.AddRange(Process.GetProcessesByName("Antigravity"));
    processes.AddRange(Process.GetProcessesByName("Antigravity Helper"));
#elif UNITY_EDITOR_LINUX
    processes.AddRange(Process.GetProcessesByName("antigravity"));
    processes.AddRange(Process.GetProcessesByName("Antigravity"));
#else
    processes.AddRange(Process.GetProcessesByName("antigravity"));
#endif

    foreach (var process in processes)
    {
        try
        {
            var workspaces = ProcessRunner.GetProcessWorkspaces(process);
            // ... 워크스페이스 비교 로직 (동일) ...
        }
        catch (Exception ex)
        {
            Debug.LogError($"[Antigravity] Error checking process: {ex}");
            continue;
        }
    }
    return null;
}
```

**주의**: 
- 메서드명 변경: `FindRunningCursorWithSolution` → `FindRunningAntigravityWithSolution`
- 프로세스 이름을 사전 확인한 실제 이름으로 수정
- 로그 메시지 변경

#### 2.7 Open 메서드 (578-617줄)

**변경 전**:
```csharp
public override bool Open(string path, int line, int column, string solution)
{
    // ... 초기화 코드 ...

    if (EditorPrefs.GetBool(ReuseExistingWindowKey, false))
    {
        var existingProcess = FindRunningCursorWithSolution(directory);
        if (existingProcess != null)
        {
            try
            {
                var args = string.IsNullOrEmpty(path) ?
                    $"--reuse-window \"{directory}\"" :
                    $"--reuse-window -g \"{path}\":{line}:{column}";

                ProcessRunner.Start(ProcessStartInfoFor(application, args));
                return true;
            }
            catch (Exception ex)
            {
                Debug.LogError($"[Cursor] Error using existing instance: {ex}");
            }
        }
    }

    var newArgs = string.IsNullOrEmpty(path) ?
        $"--new-window \"{directory}\"" :
        $"--new-window \"{directory}\" -g \"{path}\":{line}:{column}";

    ProcessRunner.Start(ProcessStartInfoFor(application, newArgs));
    return true;
}
```

**변경 후**:
```csharp
public override bool Open(string path, int line, int column, string solution)
{
    // ... 초기화 코드 (동일) ...

    if (EditorPrefs.GetBool(ReuseExistingWindowKey, false))
    {
        var existingProcess = FindRunningAntigravityWithSolution(directory);
        if (existingProcess != null)
        {
            try
            {
                var args = string.IsNullOrEmpty(path) ?
                    $"--reuse-window \"{directory}\"" :
                    $"--reuse-window -g \"{path}\":{line}:{column}";

                ProcessRunner.Start(ProcessStartInfoFor(application, args));
                return true;
            }
            catch (Exception ex)
            {
                Debug.LogError($"[Antigravity] Error using existing instance: {ex}");
            }
        }
    }

    var newArgs = string.IsNullOrEmpty(path) ?
        $"--new-window \"{directory}\"" :
        $"--new-window \"{directory}\" -g \"{path}\":{line}:{column}";

    ProcessRunner.Start(ProcessStartInfoFor(application, newArgs));
    return true;
}
```

**변경 항목**:
- 메서드 호출: `FindRunningCursorWithSolution` → `FindRunningAntigravityWithSolution`
- 로그 메시지 변경

---

### 3. Editor/ProcessRunner.cs

**파일 위치**: `Editor/ProcessRunner.cs`

#### 3.1 GetProcessWorkspaces 메서드 (119-209줄)

**변경 전**:
```csharp
public static string[] GetProcessWorkspaces(Process process)
{
    if (process == null)
        return null;

    try
    {
        var workspaces = new List<string>();
        var userProfile = Environment.GetFolderPath(Environment.SpecialFolder.UserProfile);
        string cursorStoragePath;

#if UNITY_EDITOR_OSX
        cursorStoragePath = Path.Combine(userProfile, "Library", "Application Support", "cursor", "User", "workspaceStorage");
#elif UNITY_EDITOR_LINUX
        cursorStoragePath = Path.Combine(userProfile, ".config", "Cursor", "User", "workspaceStorage");
#else
        cursorStoragePath = Path.Combine(userProfile, "AppData", "Roaming", "cursor", "User", "workspaceStorage");
#endif
        
        if (Directory.Exists(cursorStoragePath))
        {
            // ... 워크스페이스 파싱 로직 ...
        }
        else
        {
            Debug.LogWarning($"[Cursor] Workspace storage directory not found: {cursorStoragePath}");
        }

        return workspaces.Distinct().ToArray();
    }
    catch (Exception ex)
    {
        Debug.LogError($"[Cursor] Error getting workspace directory: {ex.Message}");
        return null;
    }
}
```

**변경 후**:
```csharp
public static string[] GetProcessWorkspaces(Process process)
{
    if (process == null)
        return null;

    try
    {
        var workspaces = new List<string>();
        var userProfile = Environment.GetFolderPath(Environment.SpecialFolder.UserProfile);
        string antigravityStoragePath;

#if UNITY_EDITOR_OSX
        antigravityStoragePath = Path.Combine(userProfile, "Library", "Application Support", "antigravity", "User", "workspaceStorage");
#elif UNITY_EDITOR_LINUX
        antigravityStoragePath = Path.Combine(userProfile, ".config", "Antigravity", "User", "workspaceStorage");
#else
        antigravityStoragePath = Path.Combine(userProfile, "AppData", "Roaming", "antigravity", "User", "workspaceStorage");
#endif
        
        if (Directory.Exists(antigravityStoragePath))
        {
            // ... 워크스페이스 파싱 로직 (동일) ...
        }
        else
        {
            Debug.LogWarning($"[Antigravity] Workspace storage directory not found: {antigravityStoragePath}");
        }

        return workspaces.Distinct().ToArray();
    }
    catch (Exception ex)
    {
        Debug.LogError($"[Antigravity] Error getting workspace directory: {ex.Message}");
        return null;
    }
}
```

**주의**: 
- 변수명: `cursorStoragePath` → `antigravityStoragePath`
- 경로를 사전 확인한 실제 경로로 수정 (대소문자 주의)
- 로그 메시지 변경
- 워크스페이스 파싱 로직은 동일하게 유지 (192줄의 `Debug.LogWarning`도 변경)

---

### 4. Editor/Discovery.cs

**파일 위치**: `Editor/Discovery.cs`

**변경 전**:
```csharp
internal static class Discovery
{
    public static IEnumerable<IVisualStudioInstallation> GetVisualStudioInstallations()
    {
        foreach (var installation in VisualStudioCursorInstallation.GetVisualStudioInstallations())
            yield return installation;
        foreach (var installation in VisualStudioCodiumInstallation.GetVisualStudioInstallations())
            yield return installation;
    }

    public static bool TryDiscoverInstallation(string editorPath, out IVisualStudioInstallation installation)
    {
        try
        {
            if (VisualStudioCursorInstallation.TryDiscoverInstallation(editorPath, out installation))
                return true;
            if (VisualStudioCodiumInstallation.TryDiscoverInstallation(editorPath, out installation))
                return true;
        }
        catch (IOException)
        {
            installation = null;
        }

        return false;
    }

    public static void Initialize()
    {
        VisualStudioCursorInstallation.Initialize();
        VisualStudioCodiumInstallation.Initialize();
    }
}
```

**변경 후**:
```csharp
internal static class Discovery
{
    public static IEnumerable<IVisualStudioInstallation> GetVisualStudioInstallations()
    {
        foreach (var installation in VisualStudioAntigravityInstallation.GetVisualStudioInstallations())
            yield return installation;
        foreach (var installation in VisualStudioCodiumInstallation.GetVisualStudioInstallations())
            yield return installation;
    }

    public static bool TryDiscoverInstallation(string editorPath, out IVisualStudioInstallation installation)
    {
        try
        {
            if (VisualStudioAntigravityInstallation.TryDiscoverInstallation(editorPath, out installation))
                return true;
            if (VisualStudioCodiumInstallation.TryDiscoverInstallation(editorPath, out installation))
                return true;
        }
        catch (IOException)
        {
            installation = null;
        }

        return false;
    }

    public static void Initialize()
    {
        VisualStudioAntigravityInstallation.Initialize();
        VisualStudioCodiumInstallation.Initialize();
    }
}
```

**변경 항목**:
- 16줄: `VisualStudioCursorInstallation` → `VisualStudioAntigravityInstallation`
- 26줄: `VisualStudioCursorInstallation` → `VisualStudioAntigravityInstallation`
- 41줄: `VisualStudioCursorInstallation` → `VisualStudioAntigravityInstallation`

---

### 5. Editor/VisualStudioEditor.cs

**파일 위치**: `Editor/VisualStudioEditor.cs`

#### 5.1 OnGUI 메서드 (135-143줄)

**변경 전**:
```csharp
if (installation is VisualStudioCursorInstallation)
{
    var reuseWindow = EditorPrefs.GetBool(VisualStudioCursorInstallation.ReuseExistingWindowKey, false);
    var newReuseWindow = EditorGUILayout.Toggle(new GUIContent("Reuse existing Cursor window", "When enabled, opens files in an existing Cursor window if found. When disabled, always opens a new window."), reuseWindow);
    if (newReuseWindow != reuseWindow)
        EditorPrefs.SetBool(VisualStudioCursorInstallation.ReuseExistingWindowKey, newReuseWindow);
    
    EditorGUILayout.Space();
}
```

**변경 후**:
```csharp
if (installation is VisualStudioAntigravityInstallation)
{
    var reuseWindow = EditorPrefs.GetBool(VisualStudioAntigravityInstallation.ReuseExistingWindowKey, false);
    var newReuseWindow = EditorGUILayout.Toggle(new GUIContent("Reuse existing Antigravity window", "When enabled, opens files in an existing Antigravity window if found. When disabled, always opens a new window."), reuseWindow);
    if (newReuseWindow != reuseWindow)
        EditorPrefs.SetBool(VisualStudioAntigravityInstallation.ReuseExistingWindowKey, newReuseWindow);
    
    EditorGUILayout.Space();
}
```

**변경 항목**:
- 클래스 타입 체크: `VisualStudioCursorInstallation` → `VisualStudioAntigravityInstallation`
- UI 텍스트: "Reuse existing Cursor window" → "Reuse existing Antigravity window"
- 툴팁 텍스트: "Cursor" → "Antigravity"

---

### 6. README.md

**파일 위치**: `README.md`

**변경 전**:
```markdown
## How to install
- Unity -> Window -> Package Manager  
- Click "+" at the top left corner  
- Add package from git URL  
- Insert `https://github.com/boxqkrtm/com.unity.ide.cursor.git`  
- Add  
- Done

> **Important Notice for Users Updating from Older Versions**  
> Starting from version **v2.0.24**, the package name has been changed from  
> `com.unity.ide.cursor` to `com.boxqkrtm.ide.cursor` to prevent potential issues with Unity regarding attribution.  
> Violating these attribution rules may trigger warnings in Unity.  
> If you experience errors during the update, please remove the existing package before reinstalling the new one to avoid conflicts.
```

**변경 후**:
```markdown
# Antigravity Unity Editor Integration

Unity editor integration for Antigravity. Adds support for generating csproj files for IntelliSense, auto-discovery of installations, workspace management, and more.

## Features

- ✅ Automatic Antigravity installation detection
- ✅ C# project file generation (.csproj, .sln)
- ✅ Workspace support
- ✅ Reuse existing window option
- ✅ Unity debugger integration
- ✅ Cross-platform support (Windows, macOS, Linux)

## How to Install

1. Open Unity Editor
2. Go to **Window → Package Manager**
3. Click the **"+"** button at the top left corner
4. Select **"Add package from git URL"**
5. Insert: `https://github.com/yourusername/com.unity.ide.antigravity.git`
6. Click **"Add"**
7. Done!

## Configuration

After installation:

1. Go to **Edit → Preferences → External Tools**
2. Select **Antigravity** as your External Script Editor
3. (Optional) Enable **"Reuse existing Antigravity window"** to open files in the current window

## Requirements

- Unity 2019.4 or later
- Antigravity Editor installed on your system

## Supported Platforms

- Windows
- macOS
- Linux

## Troubleshooting

### Antigravity not detected
- Ensure Antigravity is installed in the standard location
- Check if the executable is in your system PATH

### Files not opening
- Verify Antigravity is set as the default external editor in Unity preferences
- Regenerate project files: **Assets → Open C# Project**

## License

MIT License - See LICENSE.md for details

## Credits

Based on the official Unity Visual Studio Code integration package.
```

---

### 7. CHANGELOG.md

**파일 위치**: `CHANGELOG.md`

파일 상단에 새 버전 추가:

```markdown
# Code Editor Package for Antigravity

## [2.0.27] - 2026-01-01

Integration:

- Initial Antigravity support based on Cursor integration v2.0.27
- Support for multiple or single Antigravity instance options
- Workspace support
- Auto-discovery of Antigravity installations
- Cross-platform support (Windows, macOS, Linux)

## [Previous versions from Cursor]
...
```

---

## 검증 방법

### 1. 컴파일 검증

```bash
# Unity 프로젝트에서 패키지 추가 후
# Unity Console에서 컴파일 에러 확인
```

**확인 사항**:
- [ ] 컴파일 에러 없음
- [ ] 경고 메시지 없음

### 2. 설치 탐지 검증

Unity 에디터에서:
1. **Edit → Preferences → External Tools**
2. External Script Editor 드롭다운 확인
3. **Antigravity**가 목록에 표시되는지 확인

**확인 사항**:
- [ ] Antigravity가 목록에 표시됨
- [ ] 버전 정보가 올바르게 표시됨
- [ ] 설치 경로가 올바름

### 3. 프로젝트 파일 생성 검증

1. Unity에서 **Assets → Open C# Project** 실행
2. 프로젝트 루트 디렉토리 확인

**확인 사항**:
- [ ] `.sln` 파일 생성됨
- [ ] `.csproj` 파일들 생성됨
- [ ] `.vscode/` 디렉토리 생성됨
- [ ] `.vscode/settings.json` 생성됨
- [ ] `.vscode/launch.json` 생성됨
- [ ] `.vscode/extensions.json` 생성됨

### 4. 파일 열기 검증

1. Unity에서 C# 스크립트 더블클릭
2. Antigravity가 열리는지 확인

**확인 사항**:
- [ ] Antigravity가 실행됨
- [ ] 올바른 파일이 열림
- [ ] 올바른 줄 번호로 이동

### 5. 기존 창 재사용 검증

1. **Edit → Preferences → External Tools**
2. **"Reuse existing Antigravity window"** 체크
3. Antigravity를 먼저 실행하고 프로젝트 열기
4. Unity에서 다른 스크립트 열기

**확인 사항**:
- [ ] 새 창이 열리지 않음
- [ ] 기존 Antigravity 창에서 파일이 열림

### 6. 워크스페이스 검증

1. Antigravity에서 Unity 프로젝트 폴더 열기
2. Unity에서 스크립트 열기
3. 동일한 Antigravity 인스턴스에서 열리는지 확인

**확인 사항**:
- [ ] 워크스페이스가 올바르게 인식됨
- [ ] 기존 창 재사용 옵션이 작동함

---

## 문제 해결

### 문제 1: Antigravity가 목록에 표시되지 않음

**원인**: 설치 경로가 올바르지 않음

**해결방법**:
1. Antigravity 실제 설치 경로 확인
2. `Editor/VisualStudioCursorInstallation.cs`의 `GetVisualStudioInstallations` 메서드 수정
3. 경로를 실제 설치 경로로 업데이트

### 문제 2: 파일을 열 때 Antigravity가 실행되지 않음

**원인**: 프로세스 이름이 올바르지 않음

**해결방법**:
1. Antigravity 실행 후 프로세스 이름 확인
2. `Editor/VisualStudioCursorInstallation.cs`의 `FindRunningAntigravityWithSolution` 메서드 수정
3. `Process.GetProcessesByName()` 인자를 실제 프로세스 이름으로 업데이트

### 문제 3: 기존 창 재사용이 작동하지 않음

**원인**: 워크스페이스 설정 경로가 올바르지 않음

**해결방법**:
1. Antigravity의 워크스페이스 설정 저장 경로 확인
2. `Editor/ProcessRunner.cs`의 `GetProcessWorkspaces` 메서드 수정
3. 경로를 실제 설정 경로로 업데이트

### 문제 4: 컴파일 에러 발생

**원인**: 클래스명 변경이 누락됨

**해결방법**:
1. 모든 `VisualStudioCursorInstallation` 참조를 검색
2. `VisualStudioAntigravityInstallation`으로 변경
3. 특히 `Discovery.cs`와 `VisualStudioEditor.cs` 확인

### 문제 5: Unity에서 패키지를 인식하지 못함

**원인**: `package.json`의 형식 오류

**해결방법**:
1. `package.json` 파일의 JSON 형식 검증
2. `name` 필드가 올바른 형식인지 확인 (com.*.ide.*)
3. 필수 필드들이 모두 존재하는지 확인

---

## 추가 커스터마이징

### 다른 에디터 지원 제거

Codium 지원이 필요없다면:

1. `Editor/VisualStudioCodiumInstallation.cs` 삭제
2. `Discovery.cs`에서 Codium 관련 코드 제거:
```csharp
public static IEnumerable<IVisualStudioInstallation> GetVisualStudioInstallations()
{
    foreach (var installation in VisualStudioAntigravityInstallation.GetVisualStudioInstallations())
        yield return installation;
    // Codium 관련 코드 삭제
}
```

### 커스텀 설치 경로 추가

특정 경로를 추가하려면 `GetVisualStudioInstallations` 메서드에 추가:

```csharp
#if UNITY_EDITOR_WIN
    candidates.Add(@"C:\CustomPath\Antigravity\antigravity.exe");
#endif
```

---

## 체크리스트

작업 완료 전 최종 확인:

### 코드 수정
- [ ] `package.json` 수정 완료
- [ ] `Editor/VisualStudioCursorInstallation.cs` 모든 항목 수정 완료
- [ ] `Editor/ProcessRunner.cs` 수정 완료
- [ ] `Editor/Discovery.cs` 수정 완료
- [ ] `Editor/VisualStudioEditor.cs` 수정 완료
- [ ] `README.md` 수정 완료
- [ ] `CHANGELOG.md` 업데이트 완료

### 검증
- [ ] 컴파일 에러 없음
- [ ] Antigravity 자동 탐지 작동
- [ ] 프로젝트 파일 생성 작동
- [ ] 파일 열기 작동
- [ ] 기존 창 재사용 작동 (옵션 활성화 시)
- [ ] 크로스 플랫폼 테스트 (가능한 경우)

### 문서
- [ ] README.md에 설치 방법 명시
- [ ] 문제 해결 가이드 작성
- [ ] 라이선스 정보 확인

---

## 참고 자료

### Antigravity 정보 확인 명령어

#### Windows (PowerShell)
```powershell
# 실행 파일 찾기
Get-ChildItem -Path "C:\Users\$env:USERNAME\AppData\Local\Programs" -Recurse -Filter "*.exe" | Where-Object {$_.Name -like "*antigravity*"}

# 프로세스 확인
Get-Process | Where-Object {$_.ProcessName -like "*antigravity*"} | Select-Object ProcessName, Id, Path

# 설정 디렉토리 확인
Get-ChildItem "C:\Users\$env:USERNAME\AppData\Roaming" -Directory | Where-Object {$_.Name -like "*antigravity*"}
```

#### macOS/Linux (Bash)
```bash
# 실행 파일 찾기
find /Applications -name "*antigravity*" -o -name "*Antigravity*" 2>/dev/null
find /usr/local/bin -name "*antigravity*" 2>/dev/null

# 프로세스 확인
ps aux | grep -i antigravity

# 설정 디렉토리 확인 (macOS)
ls -la ~/Library/Application\ Support/ | grep -i antigravity

# 설정 디렉토리 확인 (Linux)
ls -la ~/.config/ | grep -i antigravity
```

---

## 버전 관리

### Git 저장소 설정

```bash
# 새 저장소 초기화
cd com.unity.ide.antigravity
git init

# .gitignore 설정
echo "*.meta" >> .gitignore
echo ".DS_Store" >> .gitignore
echo "Thumbs.db" >> .gitignore

# 첫 커밋
git add .
git commit -m "Initial Antigravity Unity integration"

# 원격 저장소 연결
git remote add origin https://github.com/yourusername/com.unity.ide.antigravity.git
git push -u origin main
```

### 태그 생성

```bash
git tag -a v2.0.27 -m "Initial Antigravity support"
git push origin v2.0.27
```

---

## 마무리

이 가이드를 따라 모든 수정을 완료하면 Antigravity Unity Editor 통합 패키지가 완성됩니다.

**중요**: 
- 실제 Antigravity 설치 정보를 반드시 확인하고 코드에 반영하세요
- 각 플랫폼에서 테스트하여 정상 작동을 확인하세요
- 문제가 발생하면 Unity Console과 Antigravity의 로그를 확인하세요

작업 중 문제가 발생하면 이 가이드의 "문제 해결" 섹션을 참고하세요.
