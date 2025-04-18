# PowerShell Keyboard Layout Indicator

> [Русская версия](README-ru.md) | [English version](README.md)

Displays current keyboard layout (`<en>` or `<ru>`) with color indicators directly in PowerShell prompt.

## ✅ Features

- Shows current keyboard layout status
- Color indicators:
  - 🟢 Green - English layout (`<en>`)
  - 🔵 Blue - Russian layout (`<ru>`)
- Auto-updates when layout changes
- Works in **PowerShell 7+**

---

## 🛠️ Installation

### 1. Create the script

1. Open PowerShell 7
2. Run these commands:

```powershell
# Create folder for system scripts
mkdir "$env:USERPROFILE\Documents\system" -Force

# Open script file in VS Code
code "$env:USERPROFILE\Documents\system\keyboard_prompt.ps1"
```

3. Paste this code into the opened file:

```powershell
<#
.SYNOPSIS
    Adds keyboard layout indicator to PowerShell prompt
#>

Add-Type @"
using System;
using System.Runtime.InteropServices;
public class KeyboardLayout {
    [DllImport("user32.dll")]
    private static extern IntPtr GetKeyboardLayout(uint idThread);

    [DllImport("user32.dll")]
    private static extern uint GetWindowThreadProcessId(IntPtr hWnd, out uint lpdwProcessId);

    [DllImport("user32.dll")]
    private static extern IntPtr GetForegroundWindow();

    public static string GetCurrentKeyboardLayout() {
        try {
            IntPtr foregroundWindow = GetForegroundWindow();
            uint processId;
            uint threadId = GetWindowThreadProcessId(foregroundWindow, out processId);
            IntPtr keyboardLayout = GetKeyboardLayout(threadId);
            uint localeId = (uint)keyboardLayout.ToInt32() & 0xFFFF;
            return new System.Globalization.CultureInfo((int)localeId).Name;
        } catch {
            return "en-US"; // Fallback to English layout on error
        }
    }
}
"@

function global:prompt {
    $layout = [KeyboardLayout]::GetCurrentKeyboardLayout()
    $indicator = if ($layout -like "en-*") {
        "`e[38;5;34m<en>`e[0m"  # green
    } else {
        "`e[38;5;27m<ru>`e[0m"   # blue (code 27)
    }
    "PS $($executionContext.SessionState.Path.CurrentLocation)$indicator "
}
```

---

### 2. Configure PowerShell Profile

1. Open your PowerShell profile:

```powershell
code $PROFILE
```

2. Add this line at the beginning:

```powershell
# Import keyboard layout indicator
. "$env:USERPROFILE\Documents\system\keyboard_prompt.ps1"
```

3. Save the file (Ctrl+S)

---

### 3. Apply Changes

Reload your profile with:

```powershell
. $PROFILE
```

---

## 🧪 Verification

After completing all steps, you'll see in your prompt:

```powershell
PS C:\Users\UserName<en>
```

Where:
- `<en>` - green (English layout)
- `<ru>` - blue (Russian layout)

---

## ⚠️ Notes

1. **Requirements**:
   - Windows 10/11
   - PowerShell 7.0 or newer
   - Recommended: Windows Terminal for best color display

2. **Limitations**:
   - Doesn't work in PowerShell ISE
   - May not work in Windows PowerShell 5.1

3. **Compatibility**:
   - Automatically detects active window's layout
   - Falls back to English layout on errors

---

## 🎨 Color Customization

You can change colors by modifying these codes in the script:

- `38;5;34m` - green (code 34)
- `38;5;27m` - blue (code 27)

To view all available colors:

```powershell
for ($i = 0; $i -lt 256; $i++) {
  Write-Host "`e[38;5;${i}m$i`e[0m " -NoNewline
  if (($i + 1) % 16 -eq 0) { Write-Host }
}
```

---

## 🔄 Script Updates

After modifying the script, remember to:

```powershell
. $PROFILE
```

to apply changes.
