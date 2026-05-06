# 01-18-Desarrollo conversacional con Claude Code para proyectos reales

## 01.01-contruye contigo

## 01-02-desrollar conversando

a) Pensar antes de escribir: decisiones sobre atajos.  
b) Diseñar con arquitectura existente y consistencia.  
c) Integrar GitHub, testing, performance y seguridad en el ciclo.  
d) Mantener una conversación técnica efectiva con una IA que entiende el proyecto.  
e) Asegurar calidad: pruebas y validaciones como parte del desarrollo.  
f) Enfocarte en valor de producto más que en volumen de código.

### 01-02-01-directorio de proyecto default C:\Users\AM1000897GT\.claude\projects

| Sistema operativo | Ejemplo de ruta                | Recomendación           |
| ----------------- | ------------------------------ | ----------------------- |
| Windows           | `C:\Users\AM1000897GT\.claude` | Usar `\` como separador |
| Linux             | `/home/usuario/.claude`        | Usar `/` como separador |
| macOS             | `/Users/usuario/.claude`       | Usar `/` como separador |

# 02-18-Flujo profesional para desarrollar features con Claude Code

## 02.01-acciones

1. Clona repositorio
2. Abrir carpetas
3. Revisa archivos
4. Tomar notas

## 02-02-tutorial npm

[Ver video instalacion claude](https://www.youtube.com/watch?v=A6oW7SnNq2g)

# 03-18-Instalación y configuración básica de Claude Code (manera actual)

## 03.01-macOS, Linux o WSL

Ejecuta el siguiente comando en la terminal:

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

## 03.02-Windows PowerShell

Ejecuta el siguiente comando en PowerShell:

```powershell
irm https://claude.ai/install.ps1 | iex
```

## 03.03- Windows CMD

Ejecuta el siguiente comando en CMD:

```cmd
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd
```

### 03.03-01 comando rapidos dentro de claude

Porque dentro de Claude Code hay dos tipos de interacción:

- Sin [/] → Claude lo interpreta como un mensaje o instrucción para que trabaje
- Con [/] → Es un comando del sistema de Claude Code
- ⚠️ **Desde tu terminal (fuera de Claude Code):** claude whoami.
- ⚠️ **dentro de Claude Code, usa el comando slash:** /whoami.

- <span style="color:pink"><strong>claude</strong></span>: entra al directorio de trabajo y ejecuta el comando principal para abrir el modo interactivo [REPL].
- <span style="color:red"><strong>/help</strong></span>: muestra comandos disponibles y ayuda básica.
- <span style="color:red"><strong>/status</strong></span>: reporta el estado actual del setup.
- <span style="color:red"><strong>/doctor</strong></span>: verifica la instalación y los settings. Útil para diagnosticar problemas de invocación.
- <span style="color:red"><strong>/login</strong></span>: Para utilizar los modelos necesitas iniciar sesión con login
- <span style="color:red"><strong>/claude auth status</strong> o <strong>/claude whoami</strong></span>: Te mostrará con qué cuenta estás autenticado actualmente.

## 03.04- Ventajas de usar el plugin en [IDE] vscode o cursor:

- Pasar mayor contexto desde el editor.
- Mantener conversación y código en un mismo lugar.
- Flujo de trabajo más ágil para generar y validar cambios.
- [Visitar claude-code extension](https://open-vsx.org/extension/Anthropic/claude-code)

## 03.05- Curso de tips y trucos de IA

- https://platzi.com/cursos/trucos-ia/
