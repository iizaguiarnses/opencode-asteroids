---
description: Crea un git worktree local a partir del argumento dado.
agent: build
---

Recibes un argumento: $ARGUMENTS

Tareas:

1. Analiza el argumento de arriba **tal cual** y trátalo como **un único argumento** (aunque contenga espacios, es una sola cadena; NO lo separe en varias).
2. Deriva un nombre de worktree a partir del contexto del argumento (si el argumento ya es un nombre de worktree, úsalo directamente; si es una frase, genera un slug breve y descriptivo basado en ella).
3. Ejecuta EXACTAMENTE este comando con bash, sustituyendo `<NOMBRE_DEL_WORKTREE>` por el nombre derivado en el paso 2:

```bash
git worktree add ".worktrees/<NOMBRE_DEL_WORKTREE>"
```

4. Haz **nada más**. No cambies de directorio. No hagas nada adicional. No edites archivos. Solo reporta el resultado del comando.
