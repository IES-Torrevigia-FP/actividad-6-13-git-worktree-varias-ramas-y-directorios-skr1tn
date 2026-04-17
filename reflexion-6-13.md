# Reflexión Final: Cuándo Usar `git worktree`

## 1. Ventajas de los worktrees frente a otras alternativas

### Frente a cambiar de rama en el mismo directorio

Cuando trabajas en una sola carpeta y necesitas cambiar de rama, Git obliga a hacer `checkout` o `switch`, lo que reemplaza todos los archivos del directorio por los de la otra rama. Esto significa que:

- Tienes que guardar o descartar los cambios actuales antes de cambiar (usando `stash` u otro mecanismo).
- No puedes ver ni ejecutar dos ramas al mismo tiempo.
- El proceso de cambiar de rama puede ser lento en proyectos grandes.

Con `git worktree`, cada rama tiene su propio directorio independiente. Puedes tener `main` en una terminal y `feature-a` en otra, trabajando en paralelo sin interrupciones ni necesidad de hacer stash.

### Frente a clonar el repositorio varias veces

Clonar el repositorio varias veces en diferentes carpetas parece una solución, pero tiene desventajas importantes:

- Cada clon ocupa espacio completo en disco (incluyendo toda la historia de Git duplicada).
- Los commits hechos en un clon no son visibles en los demás sin hacer `fetch` o `pull` explícito.
- Es fácil desincronizarse y perder la coherencia entre los clones.

Con `git worktree`, todos los directorios comparten el mismo `.git` y el mismo historial. Un commit hecho en cualquier worktree es instantáneamente visible en todos los demás, sin necesidad de sincronización manual.

---

## 2. Dos situaciones reales donde usaría `git worktree`

### Situación 1: Revisar un Pull Request mientras desarrollo una feature

Estoy trabajando en `feature-login` cuando me avisan de que hay un Pull Request de un compañero en la rama `feature-registro` que necesita revisión urgente. Sin worktree tendría que parar mi trabajo, hacer stash, cambiar de rama y revisar. Con worktree puedo hacer:

```bash
git worktree add ../wt-revision feature-registro
```

Abro ese directorio en otra terminal, reviso el código, ejecuto las pruebas y doy el visto bueno. Mi directorio original con `feature-login` sigue intacto, sin ningún cambio.

### Situación 2: Corregir un hotfix crítico en producción

Se detecta un bug crítico en producción (rama `main`) mientras estoy a medias de desarrollar una nueva funcionalidad en `feature-pagos`. No quiero mezclar el trabajo ni hacer stash de muchos cambios. Creo un worktree para el hotfix:

```bash
git worktree add -b hotfix-bug-critico ../wt-hotfix main
```

Corregimos el bug en `../wt-hotfix`, hacemos el commit y el despliegue. Después eliminamos ese worktree. Mi rama `feature-pagos` nunca fue tocada.

---

## 3. Buenas prácticas para nombrar y organizar los directorios de worktrees

1. **Usar un prefijo común** como `wt-` para identificar fácilmente los directorios como worktrees (ej: `wt-feature-a`, `wt-hotfix`, `wt-revision`). Así se distinguen a simple vista del directorio principal.

2. **Nombrar el directorio igual que la rama**: Si la rama se llama `feature-login`, el worktree se llama `wt-feature-login`. Esto evita confusiones sobre qué rama está en cada directorio.

3. **Crear los worktrees fuera del repositorio principal**: Usando rutas relativas como `../wt-nombre` para no meter los worktrees dentro del proyecto y que Git no los rastree como archivos del repo.

4. **Limpiar los worktrees cuando ya no se necesiten**: Ejecutar `git worktree remove <ruta>` al terminar el trabajo en una rama para no acumular directorios innecesarios. Hacer `git worktree prune` periódicamente para limpiar referencias obsoletas.

5. **Documentar los worktrees activos**: En equipos grandes, es buena práctica anotar en el ticket o PR qué worktrees se tienen activos, para que otros sepan el contexto del trabajo paralelo.
