

# EduEnroll— Plataforma mínima de matrículas, diseñada para enseñar POO moderna

> **Objetivo docente:** ejemplo integral para explicar **POO**, **herencia (ABC)**, **desacoplamiento (DIP + DI)**, **protocolos (`typing.Protocol`)**, **dataclasses**, **type hints modernos**, **listas/diccionarios** y **comprehensions**, con diseño orientado a contratos.

---

## Qué aprenderás con este repo

- **POO + dataclasses**: entidades de dominio con estado y validación.
- **Herencia y clases abstractas (ABC)**: contratos nominales para persistencia.
- **Protocolos** (`typing.Protocol`): interfaces **estructurales** (duck typing estático) para notificación y precios.
- **Desacoplamiento (DIP + DI)**: el servicio de aplicación depende de **abstracciones**, no de implementaciones.
- **Type hints de Python 3.13+**: `list[...]`, `dict[...]`, uniones con `|`, sin `Optional`, `Dict`, `List`, ni `__future__`.
- **List/dict comprehensions**: agregaciones y rankings limpios y expresivos.

---

## Piezas del dominio

- **Entidades** (`@dataclass`): `Student`, `Course`, `Enrollment`.
- **ABC** (interfaces nominales): `StudentRepository`, `EnrollmentRepository`.
- **Protocolos** (interfaces estructurales): `Notifier`, `PricingRule`.
- **Implementaciones**:
  - Repos en memoria: `InMemoryStudentRepo`, `InMemoryEnrollmentRepo`.
  - Notificadores: `EmailNotifier`, `SMSNotifier` (implementan **forma** de `Notifier`).
  - Reglas de precio: `BasePriceRule`, `CategoryDiscountRule`, `ComboRule` (implementan **forma** de `PricingRule`).
- **Servicio**: `EnrollmentService` — orquesta la matrícula **sin conocer** detalles concretos (DIP + DI).

> El diagrama de clases está en [`CLASS_DIAGRAM.md`](././CLASS_DIAGRAM.md).


Verás:

* matrícula de estudiantes,
* totales por categoría,
* top de estudiantes por gasto,
* trazas de notificación (Email/SMS).

---

## Código completo (`eduenroll.py`)

```python
from dataclasses import dataclass, field
from typing import Protocol
from abc import ABC, abstractmethod
from datetime import date

# =============== ENTIDADES (Dataclasses) ===============

@dataclass
class Student:
    id: int
    name: str
    email: str

@dataclass
class Course:
    id: int
    title: str
    category: str               # p.ej. "CS", "MATH", "AI"
    base_price: float

    def __post_init__(self) -> None:
        if self.base_price <= 0:
            raise ValueError("base_price debe ser > 0")

@dataclass
class Enrollment:
    id: int
    student_id: int
    course_id: int
    enrolled_on: date = field(default_factory=date.today)
    paid_amount: float = 0.0


# =============== ABC: Repositorios nominales ===============

class StudentRepository(ABC):
    """Contrato de persistencia para estudiantes."""
    @abstractmethod
    def add(self, student: Student) -> None: ...
    @abstractmethod
    def get(self, student_id: int) -> Student | None: ...
    @abstractmethod
    def list_all(self) -> list[Student]: ...

class EnrollmentRepository(ABC):
    """Contrato de persistencia para matrículas."""
    @abstractmethod
    def add(self, enrollment: Enrollment) -> None: ...
    @abstractmethod
    def next_id(self) -> int: ...
    @abstractmethod
    def list_by_student(self, student_id: int) -> list[Enrollment]: ...
    @abstractmethod
    def list_all(self) -> list[Enrollment]: ...


# =============== Implementaciones concretas (en memoria) ===============

class InMemoryStudentRepo(StudentRepository):
    def __init__(self) -> None:
        self._data: dict[int, Student] = {}

    def add(self, student: Student) -> None:
        if student.id in self._data:
            raise ValueError(f"Student id {student.id} already exists")
        self._data[student.id] = student

    def get(self, student_id: int) -> Student | None:
        return self._data.get(student_id)

    def list_all(self) -> list[Student]:
        return list(self._data.values())

class InMemoryEnrollmentRepo(EnrollmentRepository):
    def __init__(self) -> None:
        self._data: dict[int, Enrollment] = {}
        self._counter: int = 1

    def add(self, enrollment: Enrollment) -> None:
        if enrollment.id in self._data:
            raise ValueError(f"Enrollment id {enrollment.id} already exists")
        self._data[enrollment.id] = enrollment

    def next_id(self) -> int:
        nid = self._counter
        self._counter += 1
        return nid

    def list_by_student(self, student_id: int) -> list[Enrollment]:
        return [e for e in self._data.values() if e.student_id == student_id]

    def list_all(self) -> list[Enrollment]:
        return list(self._data.values())


# =============== Protocolos (interfaces estructurales) ===============

class Notifier(Protocol):
    """Cualquier objeto con send(to, subject, body) es un Notifier."""
    def send(self, to: str, subject: str, body: str) -> None: ...

class PricingRule(Protocol):
    """Cualquier objeto con price_for(course) es una regla de precio."""
    def price_for(self, course: Course) -> float: ...


# =============== Implementaciones que CUMPLEN los Protocolos ===============

class EmailNotifier:
    def __init__(self, sender: str = "no-reply@university.edu") -> None:
        self.sender = sender

    def send(self, to: str, subject: str, body: str) -> None:
        print(f"[EMAIL from {self.sender} → {to}] {subject}\n{body}\n")

class SMSNotifier:
    def __init__(self, provider: str = "Twilio") -> None:
        self.provider = provider

    def send(self, to: str, subject: str, body: str) -> None:
        print(f"[SMS via {self.provider} → {to}] {body}\n")


# =============== Reglas de precio (componibles) ===============

class BasePriceRule:
    """Precio base del curso (sin descuentos)."""
    def price_for(self, course: Course) -> float:
        return course.base_price

class CategoryDiscountRule:
    """
    Aplica descuento por categoría (p.ej. AI: 10%, MATH: 5%).
    """
    def __init__(self, discounts: dict[str, float]) -> None:
        # valores en [0,1], ej {"AI": 0.10, "MATH": 0.05}
        self.discounts = discounts

    def price_for(self, course: Course) -> float:
        d = self.discounts.get(course.category, 0.0)
        return course.base_price * (1 - d)

class ComboRule:
    """
    Combina varias reglas y toma el menor precio resultante (mejor oferta).
    """
    def __init__(self, rules: list[PricingRule]) -> None:
        self.rules = rules

    def price_for(self, course: Course) -> float:
        prices = [r.price_for(course) for r in self.rules]
        return min(prices) if prices else course.base_price


# =============== Servicio de aplicación (DIP + DI) ===============

class EnrollmentService:
    """
    Orquesta la matrícula:
      - Depende de ABC (repos) y Protocols (Notifier, PricingRule)
      - Se configura por inyección de dependencias
    """
    def __init__(
        self,
        student_repo: StudentRepository,
        enrollment_repo: EnrollmentRepository,
        notifier: Notifier,
        pricing: PricingRule,
    ) -> None:
        self.student_repo = student_repo
        self.enrollment_repo = enrollment_repo
        self.notifier = notifier
        self.pricing = pricing

    def enroll(self, student_id: int, course: Course) -> Enrollment:
        student = self.student_repo.get(student_id)
        if student is None:
            raise ValueError(f"Student {student_id} not found")

        enrollment_id = self.enrollment_repo.next_id()
        price = self.pricing.price_for(course)

        enrollment = Enrollment(
            id=enrollment_id,
            student_id=student.id,
            course_id=course.id,
            paid_amount=price,
        )
        self.enrollment_repo.add(enrollment)

        subject = f"Matriculado en {course.title}"
        body = (
            f"Hola {student.name},\n"
            f"Has sido matriculado en '{course.title}' ({course.category}).\n"
            f"Total pagado: ${price:.2f}.\n"
            f"Fecha: {enrollment.enrolled_on.isoformat()}"
        )
        self.notifier.send(student.email, subject, body)

        return enrollment


# =============== DEMO (ensamblaje por inyección) ===============

if __name__ == "__main__":
    students: list[Student] = [
        Student(1, "Ana", "ana@uni.edu"),
        Student(2, "Luis", "luis@uni.edu"),
        Student(3, "Sofía", "sofia@uni.edu"),
    ]

    courses: list[Course] = [
        Course(101, "Intro a Programación", "CS", 120.0),
        Course(102, "Cálculo I", "MATH", 150.0),
        Course(201, "Introducción a IA", "AI", 220.0),
        Course(202, "Álgebra Lineal", "MATH", 180.0),
    ]

    srepo = InMemoryStudentRepo()
    erepo = InMemoryEnrollmentRepo()
    for s in students:
        srepo.add(s)

    notifier: Notifier = EmailNotifier(sender="secretaria@uni.edu")
    # notifier = SMSNotifier(provider="ACME-SMS")

    base_rule = BasePriceRule()
    cat_rule = CategoryDiscountRule(discounts={"AI": 0.15, "MATH": 0.05})
    pricing: PricingRule = ComboRule([base_rule, cat_rule])

    service = EnrollmentService(srepo, erepo, notifier, pricing)

    e1 = service.enroll(1, courses[0])
    e2 = service.enroll(2, courses[2])
    e3 = service.enroll(3, courses[1])
    e4 = service.enroll(1, courses[3])

    course_index: dict[int, Course] = {c.id: c for c in courses}
    all_enrolls = erepo.list_all()

    cats = {course_index[e.course_id].category for e in all_enrolls}
    total_by_cat: dict[str, float] = {
        cat: sum(
            e.paid_amount
            for e in all_enrolls
            if course_index[e.course_id].category == cat
        )
        for cat in cats
    }

    spent_by_student: dict[int, float] = {}
    for e in all_enrolls:
        spent_by_student[e.student_id] = spent_by_student.get(e.student_id, 0.0) + e.paid_amount
    top_students = sorted(
        spent_by_student.items(), key=lambda kv: kv[1], reverse=True
    )[:3]

    def fmt_enrollment(e: Enrollment) -> str:
        s = srepo.get(e.student_id)
        c = course_index[e.course_id]
        name = s.name if s else f"#{e.student_id}"
        return f"[{e.id}] {name} → {c.title} ({c.category}) = ${e.paid_amount:.2f}"

    print("=== MATRÍCULAS ===")
    for e in all_enrolls:
        print(" •", fmt_enrollment(e))

    print("\n=== TOTAL POR CATEGORÍA ===")
    for cat, total in total_by_cat.items():
        print(f" - {cat}: ${total:.2f}")

    print("\n=== TOP ESTUDIANTES POR GASTO ===")
    for sid, total in top_students:
        s = srepo.get(sid)
        print(f" - {(s.name if s else f'#{sid}')}: ${total:.2f}")
```

---

## Retos rápidos

1. **Nueva regla**: `EarlyBirdRule` (20% si `course.id < 200`) y combínala en `ComboRule`.
2. **Repositorio de cursos (ABC)** y úsalo en un reporte dentro de `EnrollmentService`.
3. **Nuevo protocolo** `ReceiptRenderer` con `render(enrollment, student, course) -> str` y dos implementaciones (texto/JSON).

---
