
# Diagrama de clases — EduEnroll

```mermaid
classDiagram
    %% ===== Entidades =====
    class Student {
      +id: int
      +name: str
      +email: str
    }

    class Course {
      +id: int
      +title: str
      +category: str
      +base_price: float
      +__post_init__(): None
    }

    class Enrollment {
      +id: int
      +student_id: int
      +course_id: int
      +enrolled_on: date
      +paid_amount: float
    }

    %% ===== ABC (interfaces nominales) =====
    class StudentRepository {
      +add(student: Student): None
      +get(student_id: int): Student | None
      +list_all(): list[Student]
    }
    <<abstract>> StudentRepository

    class EnrollmentRepository {
      +add(enrollment: Enrollment): None
      +next_id(): int
      +list_by_student(student_id: int): list[Enrollment]
      +list_all(): list[Enrollment]
    }
    <<abstract>> EnrollmentRepository

    %% ===== Implementaciones de repos =====
    class InMemoryStudentRepo {
      -_data: dict[int, Student]
      +add(student: Student): None
      +get(student_id: int): Student | None
      +list_all(): list[Student]
    }

    class InMemoryEnrollmentRepo {
      -_data: dict[int, Enrollment]
      -_counter: int
      +add(enrollment: Enrollment): None
      +next_id(): int
      +list_by_student(student_id: int): list[Enrollment]
      +list_all(): list[Enrollment]
    }

    StudentRepository <|-- InMemoryStudentRepo
    EnrollmentRepository <|-- InMemoryEnrollmentRepo

    %% ===== Protocolos (interfaces estructurales) =====
    class Notifier {
      +send(to: str, subject: str, body: str): None
    }
    <<interface>> Notifier

    class PricingRule {
      +price_for(course: Course): float
    }
    <<interface>> PricingRule

    %% ===== Implementaciones de Protocols =====
    class EmailNotifier {
      +sender: str
      +send(to: str, subject: str, body: str): None
    }
    class SMSNotifier {
      +provider: str
      +send(to: str, subject: str, body: str): None
    }
    EmailNotifier ..|> Notifier
    SMSNotifier ..|> Notifier

    class BasePriceRule {
      +price_for(course: Course): float
    }
    class CategoryDiscountRule {
      +discounts: dict[str, float]
      +price_for(course: Course): float
    }
    class ComboRule {
      +rules: list[PricingRule]
      +price_for(course: Course): float
    }
    BasePriceRule ..|> PricingRule
    CategoryDiscountRule ..|> PricingRule
    ComboRule ..|> PricingRule

    %% multiplicidad de ComboRule → PricingRule
    ComboRule "1" o-- "1..*" PricingRule : rules

    %% ===== Servicio =====
    class EnrollmentService {
      +student_repo: StudentRepository
      +enrollment_repo: EnrollmentRepository
      +notifier: Notifier
      +pricing: PricingRule
      +enroll(student_id: int, course: Course): Enrollment
    }

    %% Dependencias del servicio (DIP)
    EnrollmentService --> StudentRepository : depende de
    EnrollmentService --> EnrollmentRepository : depende de
    EnrollmentService --> Notifier : usa
    EnrollmentService --> PricingRule : usa

    %% Relaciones de dominio (Enrollment referencia por IDs)
    Enrollment ..> Student : ref student_id
    Enrollment ..> Course : ref course_id

    %% Repos administran entidades
    InMemoryStudentRepo "1" o-- "0..*" Student : gestiona
    InMemoryEnrollmentRepo "1" o-- "0..*" Enrollment : gestiona
````

---

## Leyenda y explicación de relaciones

> **Nota:** GitHub usa Mermaid. Los símbolos y estilos abajo siguen la sintaxis de `classDiagram`.

### Generalización / Herencia nominal

* **Símbolo:** `A <|-- B` (línea continua con **triángulo hueco** apuntando a la superclase)
* **Significado UML:** *B hereda de A* (subtipo nominal).
* **En el diagrama:**

  * `InMemoryStudentRepo` **hereda** de `StudentRepository`.
  * `InMemoryEnrollmentRepo` **hereda** de `EnrollmentRepository`.
* **Por qué:** son implementaciones concretas de las **ABC** (clases abstractas) — contrato nominal.

### Realización de interfaz (implementación de contrato)

* **Símbolo:** `A ..|> I` (línea **discontinua** con **triángulo hueco** hacia la interfaz)
* **Significado UML:** *A realiza I* (implementa la interfaz).
* **En el diagrama:**

  * `EmailNotifier ..|> Notifier`
  * `SMSNotifier ..|> Notifier`
  * Reglas de precio (`BasePriceRule`, `CategoryDiscountRule`, `ComboRule`) **realizan** `PricingRule`.
* **Por qué:** son clases que **cumplen la forma** del **protocolo** (contrato estructural). Se usa línea discontinua para diferenciar interfaz/protocolo de herencia nominal.

### Dependencia (usa / conoce)

* **Símbolo:** `A --> B` (flecha **sólida**, línea continua)
* **Significado UML:** *A depende de B* (A usa B, acoplamiento débil).
* **En el diagrama:**

  * `EnrollmentService --> StudentRepository`
  * `EnrollmentService --> EnrollmentRepository`
  * `EnrollmentService --> Notifier`
  * `EnrollmentService --> PricingRule`
* **Por qué:** el servicio **depende de abstracciones** (DIP/DI). No se modela como herencia; es una **colaboración**/uso.

### Asociación (gestiona / contiene referencias)

* **Símbolo:** `A o-- B` (**agregación**: diamante **hueco** en A)
* **Significado UML:** *A agrupa/posee B* sin ciclo de vida estricto (B puede existir sin A).
* **En el diagrama:**

  * `InMemoryStudentRepo "1" o-- "0..*" Student : gestiona`
  * `InMemoryEnrollmentRepo "1" o-- "0..*" Enrollment : gestiona`
  * `ComboRule "1" o-- "1..*" PricingRule : rules`
* **Por qué:** los repos **administran** colecciones en memoria (los objetos pueden existir fuera). `ComboRule` **agrega** varias reglas; no es composición fuerte.

> **Composición** (diamante **relleno** `*--`) **no se usa** aquí porque el ciclo de vida de los elementos **no** está estrictamente atado al contenedor (podrías reutilizar reglas o entidades en distintos contextos).

### Dependencia por referencia (IDs) / Asociación ligera

* **Símbolo:** `A ..> B` (flecha **discontinua** hacia B)
* **Significado UML:** *A conoce o hace referencia a B* de forma débil (por ID, lookup).
* **En el diagrama:**

  * `Enrollment ..> Student : ref student_id`
  * `Enrollment ..> Course : ref course_id`
* **Por qué:** `Enrollment` no contiene `Student`/`Course` completos; guarda **IDs** y resuelve afuera (p. ej., con repos). Es una **dependencia ligera**, por eso **discontinua**.

---

### Resumen visual rápido

* **Herencia nominal:** `B <|-- A` — línea **continua**, triángulo **hueco**.
* **Realización (interfaz/protocolo):** `B ..|> I` — línea **discontinua**, triángulo **hueco**.
* **Dependencia (usa):** `A --> B` — flecha **sólida** simple.
* **Agregación:** `A o-- B` — **diamante hueco** en el agregador.
* **Composición (no usada aquí):** `A *-- B` — **diamante relleno** (ciclo de vida atado).
* **Referencia débil / dependencia por ID:** `A ..> B` — flecha **discontinua** (no hay atributo directo ni fuerte).

---

## `AGGREGATION_vs_COMPOSITION.md`

# Agregación vs. Composición (mini-guía visual)

A continuación, dos mini-ejemplos para distinguir **agregación** (diamante **hueco**) y **composición** (diamante **relleno**) en UML/Mermaid.

---

## 1) Agregación — Relación débil (diamante hueco)

**Idea:** El contenedor *agrega* elementos que **pueden existir por sí mismos** y ser compartidos o movidos.

- **Ejemplo:** `Curso` agrega `Material` (PDFs, enlaces, vídeos).  
  Los `Material` **pueden existir** sin el `Curso` y reusarse en otros cursos.

```mermaid
classDiagram
    class Curso {
      +id: int
      +titulo: str
      +agregarMaterial(m: Material): void
    }
    class Material {
      +id: int
      +titulo: str
      +url: str
    }

    Curso "1" o-- "0..*" Material : agrega
````

**Cuándo usar:** cuando la vida de las partes **no depende** del todo; los materiales pueden vivir y referenciarse sin que el curso exista.

---

## 2) Composición — Relación fuerte (diamante relleno)

**Idea:** El contenedor **posee** las partes y **controla su ciclo de vida**. Si el contenedor muere, **sus partes también**.

* **Ejemplo:** `Pedido` compone `LineaPedido`.
  Una `LineaPedido` **no tiene sentido** sin su `Pedido` propietario.

```mermaid
classDiagram
    class Pedido {
      +id: int
      +fecha: date
      +agregarLinea(l: LineaPedido): void
      +total(): float
    }
    class LineaPedido {
      +producto: str
      +cantidad: int
      +precioUnitario: float
      +subtotal(): float
    }

    Pedido "1" *-- "1..*" LineaPedido : compone
```

**Cuándo usar:** cuando las partes están **fuertemente ligadas** al todo y **no** deben existir fuera de él.

---

## Pistas rápidas para clase

* **Diamante hueco `o--` (Agregación):** “Tiene-una, pero **independiente**”.
  Ej: `Equipo o-- Jugador` (el jugador puede pasar a otro equipo o existir sin equipo).

* **Diamante relleno `*--` (Composición):** “Parte-de, con **ciclo de vida atado**”.
  Ej: `Casa *-- Habitación` (si desaparece la casa, desaparecen esas habitaciones).

* **Tip práctico:** si puedes reasignar/compartir la parte fácilmente (y tiene identidad propia), suena a **agregación**.
  Si la parte **nace y muere** con el todo, suena a **composición**.

---
