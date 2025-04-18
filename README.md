# PowerShell-Keyboard-Layout-Indicator
Индикатор раскладки клавиатуры в PowerShell

Этот скрипт отображает текущую раскладку клавиатуры (`<en>` или `<ru>`) с цветовой индикацией прямо в приглашении PowerShell.

## ✅ Возможности

- Отображение текущей раскладки клавиатуры
- Цветная индикация:
  - 🟢 Зеленый — английская раскладка (`<en>`)
  - 🔵 Синий — русская раскладка (`<ru>`)
- Автоматическое обновление при смене раскладки
- Работает в **PowerShell 7 и выше**

---

## 🛠️ Установка

### 1. Создание скрипта

1. Откройте PowerShell 7
2. Выполните команды:

```powershell
# Создаем папку для системных скриптов
mkdir "$env:USERPROFILE\Documents\system" -Force

# Открываем файл скрипта в VS Code
code "$env:USERPROFILE\Documents\system\keyboard_prompt.ps1"
```

3. Вставьте следующий код в открывшийся файл:

```powershell
<#
.SYNOPSIS
    Добавляет индикатор раскладки клавиатуры в приглашение PowerShell
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
            return "en-US"; // Возвращаем английскую раскладку по умолчанию в случае ошибки
        }
    }
}
"@

function global:prompt {
    $layout = [KeyboardLayout]::GetCurrentKeyboardLayout()
    $indicator = if ($layout -like "en-*") {
        "`e[38;5;34m<en>`e[0m"  # зелёный
    } else {
        "`e[38;5;27m<ru>`e[0m"   # синий (код 27)
    }
    "PS $($executionContext.SessionState.Path.CurrentLocation)$indicator "
}
```

---

### 2. Настройка профиля PowerShell

1. Откройте файл профиля PowerShell:

```powershell
code $PROFILE
```

2. Добавьте в начало файла строку:

```powershell
# Подключение индикатора раскладки клавиатуры
. "$env:USERPROFILE\Documents\system\keyboard_prompt.ps1"
```

3. Сохраните файл (Ctrl+S)

---

### 3. Применение изменений

Выполните команду для обновления профиля:

```powershell
. $PROFILE
```

---

## 🧪 Проверка работы

После выполнения всех шагов в приглашении PowerShell появится индикатор:

```powershell
PS C:\Users\UserName<en>
```

Где:
- `<en>` — зелёный цвет для английской раскладки
- `<ru>` — синий цвет для русской раскладки

---

## ⚠️ Примечания

1. **Требования**:
   - Windows 10/11
   - PowerShell 7.0 или новее
   - Рекомендуется использовать Windows Terminal для лучшего отображения цветов

2. **Ограничения**:
   - Не работает в PowerShell ISE
   - Может не работать в Windows PowerShell 5.1

3. **Совместимость**:
   - Скрипт автоматически определяет раскладку активного окна
   - В случае ошибки показывает английскую раскладку по умолчанию

---

## 🎨 Настройка цветов

Вы можете изменить цвета индикаторов, заменив коды в скрипте:

- `38;5;34m` — зеленый (код 34)
- `38;5;27m` — синий (код 27)

Для просмотра всех доступных цветов выполните:

```powershell
for ($i = 0; $i -lt 256; $i++) {
  Write-Host "`e[38;5;${i}m$i`e[0m " -NoNewline
  if (($i + 1) % 16 -eq 0) { Write-Host }
}
```

---

## 🔄 Обновление скрипта

При изменении скрипта не забудьте выполнить:

```powershell
. $PROFILE
```

для применения изменений.
