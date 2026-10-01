# LimpiezaCarpetasUsuario — Tarea Programada de PowerShell

Script de automatización para Windows que registra una tarea programada en el Programador de Tareas (*Task Scheduler*). Su objetivo es mantener limpios los perfiles de usuario eliminando archivos temporales o descargados al iniciar sesión, preservando los accesos directos del escritorio.

---

## 📋 Descripción

El comando crea una tarea global llamada **`LimpiezaCarpetasUsuario`** configurada a nivel de grupo (`BUILTIN\Users`), ejecutándose cada vez que cualquier usuario inicia sesión en el equipo.

### ¿Qué hace exactamente?
1. **Descargas (`$HOME\Downloads`):** Elimina de forma recursiva y forzada todo el contenido.
2. **Documentos (`$HOME\Documents`):** Elimina de forma recursiva y forzada todo el contenido.
3. **Escritorio (`$HOME\Desktop`):** Elimina todos los archivos y subcarpetas, **exceptuando**:
   - Accesos directos estándar (`*.lnk`).
   - Accesos directos de internet (`*.url`).

---

## ⚙️ Comando de Instalación

Ejecuta el siguiente comando en una consola de **PowerShell con privilegios de Administrador**:

```powershell
Register-ScheduledTask -TaskName "LimpiezaCarpetasUsuario" `
  -Trigger (New-ScheduledTaskTrigger -AtLogOn) `
  -Action (New-ScheduledTaskAction -Execute "powershell.exe" -Argument '-WindowStyle Hidden -Command "Remove-Item -Path \"$HOME\Downloads\*\", \"$HOME\Documents\*\" -Recurse -Force -ErrorAction SilentlyContinue; Get-ChildItem -Path \"$HOME\Desktop\" -Exclude *.lnk, *.url | Remove-Item -Recurse -Force -ErrorAction SilentlyContinue"') `
  -Principal (New-ScheduledTaskPrincipal -GroupId "BUILTIN\Users") `
  -Force
```

---

## 🔍 Parámetros y Componentes

| Parámetro / Componente | Descripción |
| :--- | :--- |
| `-TaskName "LimpiezaCarpetasUsuario"` | Nombre identificador de la tarea programada. |
| `-Trigger (New-ScheduledTaskTrigger -AtLogOn)` | Desencadenador: se activa automáticamente cuando el usuario inicia sesión. |
| `-Action (New-ScheduledTaskAction ...)` | Acción a ejecutar: lanza una instancia invisible (`-WindowStyle Hidden`) de PowerShell. |
| `-Principal (New-ScheduledTaskPrincipal -GroupId "BUILTIN\Users")` | Permite que la tarea aplique para todos los usuarios estándar o locales del grupo `BUILTIN\Users`. |
| `-Force` | Sobrescribe cualquier tarea previa con el mismo nombre sin pedir confirmación interactiva. |
| `-ErrorAction SilentlyContinue` | Ignora advertencias o errores si las rutas no existen o los archivos están bloqueados/en uso. |

---

## ⚠️ Consideraciones de Seguridad e Impacto

> [!WARNING]
> **Eliminación permanente:** `Remove-Item` **no** envía los archivos a la Papelera de Reciclaje; los destruye permanentemente. Asegúrate de que los usuarios no guarden información crítica en las rutas afectadas sin un respaldo previo.

- **Equipos compartidos o quioscos:** Esta tarea es ideal para aulas, cibercafés, terminales de autoservicio o laboratorios donde los perfiles deben restaurarse a un estado limpio tras cada sesión.
- **Archivos en uso:** Si un archivo está abierto por otro proceso en el momento del inicio de sesión, `Remove-Item` fallará silenciosamente sin interrumpir el resto de la limpieza.

---

## 🛠️ Comprobación y Administración

### 1. Verificar el registro de la tarea
Para comprobar que la tarea quedó registrada correctamente:
```powershell
Get-ScheduledTask -TaskName "LimpiezaCarpetasUsuario"
```

### 2. Probar ejecución manual
Puedes forzar su ejecución sin necesidad de cerrar sesión:
```powershell
Start-ScheduledTask -TaskName "LimpiezaCarpetasUsuario"
```

### 3. Desinstalar / Eliminar la tarea
Si deseas deshabilitar o eliminar la limpieza automática:
```powershell
Unregister-ScheduledTask -TaskName "LimpiezaCarpetasUsuario" -Confirm:$false
```
