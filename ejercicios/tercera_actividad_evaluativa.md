# Recetario de cocina

En el módulo `receta.py`, completa la aplicación siguiendo estos pasos:

1. Cree una clase `Receta` con los atributos:

   * `code_id` (int), `nombre` (str), `instrucciones` (str), `preparada` (bool), `etiquetas` (list).
   * En el constructor, reciba `code_id`, `nombre`, `instrucciones`.
   * Inicialice `preparada` en `False` y `etiquetas` como `[]`.

2. En `Receta`, defina un método de instancia `marcar_preparada()` que cambie `preparada` a `True`.

3. En `Receta`, defina `agregar_etiqueta(tag: str)` que agregue `tag` a `etiquetas` **solo si no existe**.

4. En `Receta`, defina `__str__(self) -> str` que retorne `"code_id - nombre"`.

5. Cree una clase `Recetario` con un atributo `recetas` que sea un diccionario donde la clave es `int` y el valor es un objeto `Receta`. Inícielo como `{}`.

6. En `Recetario`, defina `agregar_receta(nombre: str, instrucciones: str) -> int` que:

   * Genere un id como `len(self.recetas) + 1`.
   * Cree una `Receta`.
   * La agregue al diccionario `recetas` con la clave generada.
   * Retorne el id.

7. En `Recetario`, defina `recetas_pendientes() -> list` que retorne una lista con las recetas cuyo atributo `preparada` sea `False`. **No use comprensiones**.

8. En `Recetario`, defina `recetas_preparadas() -> list` que retorne una lista con las recetas cuyo atributo `preparada` sea `True`. **No use comprensiones**.

9. En `Recetario`, defina `conteo_etiquetas() -> dict` que retorne `{etiqueta: cantidad_de_recetas_que_la_tienen}` construido con bucles (**sin comprensiones**).

### Plantilla incompleta — `receta.py`

```python
class Receta:
    def __init__(self, code_id: int, nombre: str, instrucciones: str) -> None:
        # TODO: inicializar:
        # Code_id_nombre,instrucciones
        # Parada
        # Etiquetas
        pass

    def marcar_preparada(self) -> None:
        # TODO: poner preparada en True
        pass

    def agregar_etiqueta(self, tag: str) -> None:
        # TODO: agregar tag solo si NO existe en etiquetas
        pass

    def __str__(self) -> str:
        # TODO: retornar "code_id - nombre"
        pass


class Recetario:
    def __init__(self) -> None:
        # TODO: inicializar:
        # Recetas
        pass

    def agregar_receta(self, nombre: str, instrucciones: str) -> int:
        # TODO:
        # - generar nuevo_id 
        # - crear un objeto de la clase Receta
        # - guardar en recetas
        # - retornar nuevo_id
        pass

    def recetas_pendientes(self) -> list:
        # TODO: construir lista con recetas donde preparada == False 
        pass

    def recetas_preparadas(self) -> list:
        # TODO: construir lista con recetas donde preparada == True 
        pass

    def conteo_etiquetas(self) -> dict:
        # TODO: construir y retornar {etiqueta: cantidad}
        pass
```

