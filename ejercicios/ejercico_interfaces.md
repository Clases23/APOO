
# StockLite

**Objetivo:** Implementar un inventario simple con **4 clases**:
1) `Item` — entidad (`@dataclass`)
2) `ItemRepository` — contrato (ABC)
3) `InMemoryItemRepository` — implementación del contrato
4) `InventoryService` — servicio (depende del contrato, no de la implementación)

**Tecnologías/conceptos a practicar:**
- POO + **dataclasses**
- **Herencia con ABC** (contrato nominal)
- **Desacoplamiento (DIP + DI)**
- List/dict **comprehensions**
- Validaciones en `__post_init__`

## Reglas y restricciones
- Usa **Python 3.13+**.
- Maneja errores con `ValueError` y mensajes claros.

---
# Diagrama de clases — StockLite

```mermaid
classDiagram
    %% ============ Entidad ============
    class Item {
      +id: int
      +name: str
      +category: str
      +unit_price: float
      +stock: int
      +tags: list[str]
      +__post_init__(): None
    }

    %% ============ Contrato (ABC) ============
    class ItemRepository {
      +add(item: Item): None
      +get(item_id: int): Item | None
      +update(item: Item): None
      +remove(item_id: int): None
      +list_all(): list[Item]
    }
    <<abstract>> ItemRepository

    %% ============ Implementación ============
    class InMemoryItemRepository {
      -_data: dict[int, Item]
      +add(item: Item): None
      +get(item_id: int): Item | None
      +update(item: Item): None
      +remove(item_id: int): None
      +list_all(): list[Item]
    }

    %% ============ Servicio ============
    class InventoryService {
      +repo: ItemRepository
      +add_item(item: Item): None
      +restock(item_id: int, qty: int): None
      +sell(item_id: int, qty: int): None
      +items_by_category(cat: str): list[Item]
      +low_stock(threshold: int): list[Item]
      +valuation(): float
      +top_n_by_value(n: int): list[Item]
      +stats_by_category(): dict[str,int]
    }

    %% Relaciones
    ItemRepository <|-- InMemoryItemRepository
    InventoryService --> ItemRepository : depende de
    InMemoryItemRepository "1" o-- "0..*" Item : gestiona
````
## Lo que debes construir

### 1) `Item` (`@dataclass`)
Campos:
- `id: int`
- `name: str`
- `category: str`
- `unit_price: float` (debe ser `> 0`)
- `stock: int` (debe ser `>= 0`)
- `tags: list[str] = field(default_factory=list)`

En `__post_init__`:
- Validar `name/category` no vacíos (usar `.strip()`).
- Validar `unit_price > 0` y `stock >= 0`.
- Normalizar `tags` (strip, sin vacíos).

### 2) `ItemRepository` (ABC)
Contrato:
- `add(item: Item) -> None` (rechaza id duplicado)
- `get(item_id: int) -> Item | None`
- `update(item: Item) -> None` (falla si no existe)
- `remove(item_id: int) -> None` (falla si no existe)
- `list_all() -> list[Item]`

### 3) `InMemoryItemRepository`
Implementa el contrato en memoria con `dict[int, Item]`.

### 4) `InventoryService`
Depende de `ItemRepository` (inyéctalo en `__init__`).
Implementa:
- `add_item(item: Item) -> None`
- `restock(item_id: int, qty: int) -> None` (qty > 0)
- `sell(item_id: int, qty: int) -> None`  
  - `qty > 0` y `stock` suficiente; si no, `ValueError`.
- `items_by_category(cat: str) -> list[Item]` (case-insensitive)
- `low_stock(threshold: int) -> list[Item]` (items con `stock <= threshold`)
- `valuation() -> float` (∑ `unit_price * stock`)
- `top_n_by_value(n: int) -> list[Item]` (orden desc por `unit_price*stock`)
- `stats_by_category() -> dict[str, int]` (conteo de items por categoría)

---

## Pruebas sugeridas rápidas

1. **Alta y duplicado**  
   - Agrega 3 `Item`.  
   - Intenta agregar de nuevo uno con el mismo `id` → `ValueError`.

2. **Reabastecer y vender**  
   - `restock(id, 10)` y verifica `stock`.  
   - `sell(id, 3)` → stock disminuye.  
   - `sell(id, stock+1)` → `ValueError` por insuficiencia.

3. **Filtros y reportes**  
   - `items_by_category("hardware")` (con mezcla de mayúsculas).  
   - `low_stock(5)` devuelve los esperados.  
   - `valuation()` = suma de (precio×stock).  
   - `top_n_by_value(2)` ordenado correctamente.  
   - `stats_by_category()` {cat: cantidad}.

---

## Retos extra (opcionales)
- Orden estable en `top_n_by_value` (si empatan por valor, priorizar `stock` mayor).
- Validar `name` único por categoría (extra en repo/servicio).
- Persistencia simple a JSON (guardar/cargar lista de `Item`).

---

## `stocklite.py` (starter con TODOs)

```python
from dataclasses import dataclass, field
from abc import ABC, abstractmethod

# =============== 1) ENTIDAD ==================

@dataclass(slots=True)
class Item:
    # TODO: completar campos requeridos
    # id: int
    # name: str
    # category: str
    # unit_price: float
    # stock: int
    # tags: list[str] = field(default_factory=list)
    pass

    # TODO: implementar validaciones y normalización
    # - name/category no vacíos (strip)
    # - unit_price > 0, stock >= 0
    # - tags normalizados (strip, sin vacíos)
    # def __post_init__(self) -> None:
    #     ...

# =============== 2) CONTRATO (ABC) ============

class ItemRepository(ABC):
    @abstractmethod
    def add(self, item: Item) -> None: ...
    @abstractmethod
    def get(self, item_id: int) -> Item | None: ...
    @abstractmethod
    def update(self, item: Item) -> None: ...
    @abstractmethod
    def remove(self, item_id: int) -> None: ...
    @abstractmethod
    def list_all(self) -> list[Item]: ...

# =============== 3) IMPLEMENTACIÓN ============

class InMemoryItemRepository(ItemRepository):
    def __init__(self) -> None:
        # TODO: inicializar almacenamiento interno dict[int, Item]
        # self._data: dict[int, Item] = {}
        pass

    def add(self, item: Item) -> None:
        # TODO:
        # - rechazar id duplicado (ValueError)
        # - guardar el item
        pass

    def get(self, item_id: int) -> Item | None:
        # TODO: devolver el item o None si no existe
        pass

    def update(self, item: Item) -> None:
        # TODO: fallar si no existe (ValueError), luego actualizar
        pass

    def remove(self, item_id: int) -> None:
        # TODO: fallar si no existe (ValueError), luego eliminar
        pass

    def list_all(self) -> list[Item]:
        # TODO: devolver copia de los valores (list(self._data.values()))
        pass

# =============== 4) SERVICIO (DIP + DI) =======

class InventoryService:
    """
    Depende del CONTRATO ItemRepository (DIP). Inyecta la implementación (DI).
    """
    def __init__(self, repo: ItemRepository) -> None:
        # TODO: guardar la dependencia
        pass

    def add_item(self, item: Item) -> None:
        # TODO: delegar en repo.add
        pass

    def restock(self, item_id: int, qty: int) -> None:
        # TODO: qty > 0, obtener item, aumentar stock y actualizar
        pass

    def sell(self, item_id: int, qty: int) -> None:
        # TODO: qty > 0, verificar stock suficiente, disminuir y actualizar
        pass

    def items_by_category(self, cat: str) -> list[Item]:
        # TODO: filtrar por categoría (case-insensitive) con list comprehension
        pass

    def low_stock(self, threshold: int) -> list[Item]:
        # TODO: devolver items con stock <= threshold (list comprehension)
        pass

    def valuation(self) -> float:
        # TODO: ∑ unit_price * stock (sum + gen expression)
        pass

    def top_n_by_value(self, n: int) -> list[Item]:
        # TODO: ordenar por (unit_price*stock) desc y recortar
        pass

    def stats_by_category(self) -> dict[str, int]:
        # TODO: dict comprehension {cat: conteo}
        pass

# =============== DEMO (opcional para pruebas locales) =======
if __name__ == "__main__":
    # Descomenta cuando implementes todo para probar rápido

    # repo = InMemoryItemRepository()
    # inv = InventoryService(repo)

    # inv.add_item(Item(1, "Teclado", "Hardware", 20.0, 5, ["periférico"]))
    # inv.add_item(Item(2, "Mouse", "Hardware", 10.0, 12, ["periférico", "oficina"]))
    # inv.add_item(Item(3, "Cuaderno", "Papelería", 3.5, 30, ["oficina"]))

    # inv.restock(1, 10)            # Teclado: 15
    # inv.sell(2, 2)                # Mouse: 10
    # inv.sell(3, 5)                # Cuaderno: 25

    # print("Hardware:", [i.name for i in inv.items_by_category("hardware")])
    # print("Bajo stock (<=10):", [i.name for i in inv.low_stock(10)])
    # print("Valuación total:", inv.valuation())
    # print("Top 2 valor:", [i.name for i in inv.top_n_by_value(2)])
    # print("Stats por categoría:", inv.stats_by_category())
    pass
```

---
