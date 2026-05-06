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

# 04-18-Fundamentos de Claude Code: contexto, subagentes y herramientas

## 04.00 cinco conceptos que determinates

### 04-00-02 como piensan , recuerda, colabora

- ventana de contexto
- subagentes
- model context proptocol
- CLI
- context engineering

## 04.01 ¿Qué es la ventana de contexto y por qué define tu resultado?

La ventana de contexto es la memoria de trabajo de Claude Code: todo lo que puede procesar y recordar al mismo tiempo, incluyendo tu código, la conversación y los resultados. Por defecto maneja doscientos mil tokens y existe una versión de un millón de tokens, suficiente para analizar proyectos medianos o incluso grandes. Si la memoria se llena, Claude limpia lo irrelevante y conserva lo útil.

- Memoria de trabajo con código, conversación y resultados.
- Capacidad por defecto de doscientos mil tokens.
- Opción de un millón de tokens para mayor escala.
- Gestión automática del espacio para mantener lo importante.

## 04.02 ¿Cómo optimizar la memoria con referencias y sesiones?

En la práctica, el contexto se cuida con referencias en lugar de pegar grandes bloques. Si notas olvidos, reinicia el hilo y vuelve a fijar lo esencial.

- Referencia archivos con el símbolo arroba en lugar de copiar código.
- Si olvida decisiones o detalles, abre una nueva sesión.
- Vuelve a referenciar archivos clave para reenfocar la conversación.
- Mantén el intercambio limpio de información innecesaria.

## 04.03 ¿Cómo colaboran los subagentes y se mantiene el flujo modular?

Los subagentes funcionan como miembros especializados de un equipo: cada uno con su propio contexto, herramientas y conocimiento técnico. El de arquitectura analiza la estructura del sistema, el de backend implementa APIs, el de frontend construye componentes de interfaz y el de QA valida la calidad final. Así, se evita saturar la sesión principal y se trabaja de forma modular.

- Especialización por tarea para mayor precisión.
- Contextos separados que no saturan la conversación principal.
- Flujo similar al de un equipo de desarrollo real.

## 04.04 ¿Cuándo invocar cada subagente?

Elige el especialista adecuado según el objetivo. El resultado es más enfocando y eficiente.

- Agente de backend para endpoints y APIs.
- Agente de frontend para componentes de interfaz.
- Agente de QA para pruebas y validación de calidad.
- Agente de arquitectura para visión y estructura del sistema.

## 04.05 ¿Cómo se integran MCP y la CLI y qué es el context engineering?

El tercer y cuarto pilar se complementan: MCP conecta Claude Code con tus herramientas y la CLI es la interfaz donde todo sucede. Con MCP, puedes acceder a bases de datos, correr pruebas y revisar cambios; con la CLI, Claude ejecuta comandos, analiza salidas y te ayuda a interpretarlas, manteniendo el flujo: la terminal para Claude, el editor para escribir y el navegador para validar.

### 04.05-01 ¿Qué permite MCP con tus herramientas?

La integración ocurre de forma natural, como si fueran funciones nativas, tras una configuración única.

-Acceder a bases de datos para consultas.
-Ejecutar pruebas automatizadas, por ejemplo con Playwright.
-Revisar issues y pull request en GitHub.
-Interactuar con servicios como Figma o Zapier.

### 04.05-02 ¿Cómo usar la CLI sin romper el flujo?

Claude vive en la terminal, el mismo lugar donde ya usas Git, Docker o NPM.

- Ejecuta comandos con prefijo de signo de exclamación.
- Analiza resultados y ofrece interpretación útil.
  -Mantén roles claros: terminal para Claude, editor para código, navegador para - validación.

### 04.05-02 ¿Qué es el context engineering en la práctica?

No se trata de escribir la instrucción perfecta, sino de diseñar un espacio de pensamiento compartido con los datos adecuados.

- Evita instrucciones vagas que generan ruido.
- Define propósito, límites y criterios de calidad.
- Usa referencias a archivos en lugar de pegar código.
- Indica patrones a seguir o evitar y limpia lo innecesario.
