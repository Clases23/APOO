# TaskFlow

**Objetivo docente:** mostrar, en poco código, **dataclasses**, **herencia con ABC (contrato nominal)**, **desacoplamiento (DIP + DI)**, listas/dicts y comprehensions.

- **Clases (solo 4):**
  1) `Task` — entidad (`@dataclass`)
  2) `TaskRepository` — contrato (ABC)
  3) `InMemoryTaskRepository` — implementación del contrato
  4) `TaskService` — servicio de aplicación (depende del contrato, no de la implementación)



---

##  Código

```python
from dataclasses import dataclass, field
from abc import ABC, abstractmethod
from datetime import date

# =============== 1) ENTIDAD ==================

@dataclass(slots=True)
class Task:
    id: int
    title: str
    status: str = "todo"            # "todo" | "doing" | "done"
    priority: int = 3               # 1=alta, 2=media, 3=baja
    tags: list[str] = field(default_factory=list)
    created_on: date = field(default_factory=date.today)

    def __post_init__(self) -> None:
        if not self.title.strip():
            raise ValueError("title vacío")
        if self.status not in {"todo", "doing", "done"}:
            raise ValueError("status inválido")
        if self.priority not in {1, 2, 3}:
            raise ValueError("priority debe ser 1, 2 o 3")
        # normalizar tags
        self.tags = [t.strip() for t in self.tags if t.strip()]

# =============== 2) CONTRATO (ABC) ============

class TaskRepository(ABC):
    @abstractmethod
    def add(self, task: Task) -> None: ...
    @abstractmethod
    def get(self, task_id: int) -> Task | None: ...
    @abstractmethod
    def update(self, task: Task) -> None: ...
    @abstractmethod
    def remove(self, task_id: int) -> None: ...
    @abstractmethod
    def list_all(self) -> list[Task]: ...

# =============== 3) IMPLEMENTACIÓN ============

class InMemoryTaskRepository(TaskRepository):
    def __init__(self) -> None:
        self._data: dict[int, Task] = {}

    def add(self, task: Task) -> None:
        if task.id in self._data:
            raise ValueError(f"ID duplicado: {task.id}")
        self._data[task.id] = task

    def get(self, task_id: int) -> Task | None:
        return self._data.get(task_id)

    def update(self, task: Task) -> None:
        if task.id not in self._data:
            raise ValueError(f"No existe task {task.id}")
        self._data[task.id] = task

    def remove(self, task_id: int) -> None:
        if task_id not in self._data:
            raise ValueError(f"No existe task {task_id}")
        del self._data[task_id]

    def list_all(self) -> list[Task]:
        return list(self._data.values())

# =============== 4) SERVICIO (DIP + DI) =======

class TaskService:
    """
    Orquesta casos de uso.
    Depende del CONTRATO TaskRepository (DIP) e inyectamos la implementación (DI).
    """
    def __init__(self, repo: TaskRepository) -> None:
        self.repo = repo

    def create(self, task: Task) -> None:
        self.repo.add(task)

    def mark_done(self, task_id: int) -> None:
        t = self.repo.get(task_id)
        if t is None:
            raise ValueError("task no encontrada")
        t.status = "done"
        self.repo.update(t)

    def filter_by_tag(self, tag: str) -> list[Task]:
        needle = tag.lower().strip()
        return [t for t in self.repo.list_all() if needle in (x.lower() for x in t.tags)]

    def top_n_by_priority(self, n: int) -> list[Task]:
        # 1=alta primero → orden ascendente por priority
        return sorted(self.repo.list_all(), key=lambda t: t.priority)[:max(0, n)]

    def stats_by_status(self) -> dict[str, int]:
        # dict comprehension + sum en comp. generadores
        statuses = {"todo", "doing", "done"}
        tasks = self.repo.list_all()
        return {s: sum(1 for t in tasks if t.status == s) for s in statuses}

# =============== DEMO =========================

if __name__ == "__main__":
    repo = InMemoryTaskRepository()
    svc = TaskService(repo)

    svc.create(Task(1, "Instalar Python", tags=["setup", "python"], priority=1))
    svc.create(Task(2, "Leer PEPs", tags=["python", "reading"], priority=2))
    svc.create(Task(3, "Practicar dataclasses", tags=["python", "poo"], priority=1))
    svc.create(Task(4, "Commit inicial", tags=["git"], priority=3, status="doing"))

    svc.mark_done(1)

    print("Todas:")
    for t in svc.repo.list_all():
        print(" -", t)

    print("\nPor tag 'python':", [t.title for t in svc.filter_by_tag("python")])
    print("Top 2 por prioridad:", [t.title for t in svc.top_n_by_priority(2)])
    print("Stats:", svc.stats_by_status())
```

---

## Puntos para explicar en clase

* **POO + dataclass:** `Task` con estado/validación (`__post_init__`).
* **Herencia (ABC):** `TaskRepository` como contrato → `InMemoryTaskRepository` lo implementa.
* **Desacoplamiento (DIP + DI):** `TaskService` depende del contrato, se inyecta la implementación.
* **Type hints 3.13+:** `list[...]`, `dict[...]`, uniones `|`.
* **Comprehensions:** filtros, rankings y estadísticas en una línea.

# Diagrama de clases — TaskFlow (4 clases)

```mermaid
classDiagram
    class Task {
      +id: int
      +title: str
      +status: str
      +priority: int
      +tags: list[str]
      +created_on: date
      +__post_init__(): None
    }

    class TaskRepository {
      +add(task: Task): None
      +get(task_id: int): Task | None
      +update(task: Task): None
      +remove(task_id: int): None
      +list_all(): list[Task]
    }
    <<abstract>> TaskRepository

    class InMemoryTaskRepository {
      -_data: dict[int, Task]
      +add(task: Task): None
      +get(task_id: int): Task | None
      +update(task: Task): None
      +remove(task_id: int): None
      +list_all(): list[Task]
    }

    class TaskService {
      +repo: TaskRepository
      +create(task: Task): None
      +mark_done(task_id: int): None
      +filter_by_tag(tag: str): list[Task]
      +top_n_by_priority(n: int): list[Task]
      +stats_by_status(): dict[str,int]
    }

    TaskRepository <|-- InMemoryTaskRepository
    TaskService --> TaskRepository : depende de
    InMemoryTaskRepository "1" o-- "0..*" Task : gestiona
````


