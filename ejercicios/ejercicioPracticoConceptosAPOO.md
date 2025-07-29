
**Ejercicio 1 : “Tablero Kanban de tareas”**

> **Objetivo:** Que cada estudiante modele un sencillo sistema de gestión de tareas y practique la interacción entre dos clases.

1. **Crear la clase `Tarea`.**

   * Constructor `__init__(self, titulo, descripcion, estado = "Por hacer")`

     * `self.titulo`
     * `self.descripcion`
     * `self.estado`
   * Métodos:

     * `mover_estado(self, nuevo_estado)`: asigna `self.estado = nuevo_estado` y muestra `"Tarea '{titulo}' → estado: {nuevo_estado}"`.
     * `mostrar_info(self)`: imprime `"[{estado}] {titulo}: {descripcion}"`.

2. **Crear la clase `TableroKanban`.**

   * Constructor `__init__(self, nombre)`

     * `self.nombre`
     * `self.tareas = []`
   * Métodos:

     * `añadir_tarea(self, tarea: Tarea)`: añade a `self.tareas` e imprime confirmación.
     * `mostrar_tareas(self)`: recorre `self.tareas` y llama a `mostrar_info()` en cada `Tarea`.
     * `mover_tarea(self, titulo: str, nuevo_estado: str)`: busca la tarea por `titulo`, si la encuentra llama a `mover_estado`, si no, imprime aviso.

3. **Pasos de práctica (autoguiados):**

   1. Copiar las definiciones de `Tarea` y `TableroKanban` en un proyecto PyCharm.
   2. Crear tres tareas diferentes:

      ```python
      t1 = Tarea("Diseñar UI", "Definir paleta de colores y tipografías")
      t2 = Tarea("Implementar backend", "API REST con Flask")
      t3 = Tarea("Escribir tests", "Cobertura mínima 80%")
      ```
   3. Crear el tablero: `tab = TableroKanban("Proyecto A")`.
   4. Añadir las tres tareas con `tab.añadir_tarea(t1)`, etc.
   5. Llamar `tab.mostrar_tareas()` para ver estados iniciales.
   6. Ejecutar `tab.mover_tarea("Implementar backend", "En progreso")`.
   7. Volver a llamar `tab.mostrar_tareas()` y comprobar que el estado ha cambiado.

---
