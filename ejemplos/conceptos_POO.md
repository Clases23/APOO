**Temas que se van a tratar en los ejercicios**

* Definición de clases en Python
* Atributos de clase y atributos de instancia
* Métodos de instancia
* Métodos protegidos (`_metodo`)
* Métodos privados (`__metodo`)
* Métodos estáticos (`@staticmethod`)
* Métodos de clase (`@classmethod`)
* Anotaciones de tipo (type hints)

---

## Ejercicio 1: Círculo

**Enunciado:**
Modela la clase `Circulo` en Python con:

1. **Atributo de clase**

   * `pi: float` con el valor de π.
2. **Constructor** (`__init__`)

   * Parámetro `radio: float` → `self.radio`.
3. **Método de instancia** `calcular_area() -> float | None` que:

   * Llama a `_validar_radio()` (método protegido).
   * Si es válido, calcula `pi · radio²` y lo formatea con `__formatear(valor: float)` (método privado).
   * Si no, imprime un mensaje de error y devuelve `None`.
4. **Método protegido** `_validar_radio() -> bool`

   * Verifica que `radio > 0`.
5. **Método privado** `__formatear(valor: float) -> float`

   * Redondea a 2 decimales.
6. **Método estático** `convertir_grados_a_radianes(grados: float) -> float`

   * Convierte grados a radianes usando π.
7. **Método de clase** `circulo_unitario() -> Circulo`

   * Devuelve una instancia con `radio = 1.0`.

### Espacio para la solución en Python.

```python
# --- Implementa aquí la clase Circulo ---

# --- Pruebas de uso para Circulo ---
c1 = Circulo(5)
print("Área esperada (radio=5):", c1.calcular_area())           # → 78.54
print("180° →", Circulo.convertir_grados_a_radianes(180))       # → 3.14159…
u = Circulo.circulo_unitario()
print("Círculo unitario radio:", u.radio, "→ Área:", u.calcular_area())  # → 1, 3.14
c2 = Circulo(-2)
print("Prueba radio negativo:", c2.calcular_area())            # → None (y mensaje de error)
```

---

## Ejercicio 2: Conversor de Temperaturas

**Enunciado:**
Crea la clase `ConversorTemperatura` para convertir entre Celsius, Fahrenheit y Kelvin. Tu clase deberá:

1. **Atributo de clase**

   * `escala_base: str = "Celsius"`.
2. **Constructor** (`__init__`)

   * Parámetro `valor_c: float` → `self.valor_c`.
3. **Método de instancia** `a_fahrenheit() -> float | None` que:

   * Llama a `_validar_valor()` (método protegido).
   * Si es válido, convierte `C·9/5 + 32` y lo formatea con `__formatear(temp: float)` (método privado).
   * Si no, imprime mensaje de error y devuelve `None`.
4. **Método protegido** `_validar_valor() -> bool`

   * Verifica que `valor_c >= -273.15`.
5. **Método privado** `__formatear(temp: float) -> float`

   * Redondea a 1 decimal.
6. **Método estático** `celsius_a_kelvin(celsius: float) -> float`

   * Convierte `C → K` con `+ 273.15`.
7. **Método de clase** `desde_fahrenheit(fh: float) -> ConversorTemperatura`

   * Convierte `F → C` con `(F - 32)·5/9` y devuelve una instancia.

### Espacio para la solución en Python.

```python
# --- Implementa aquí la clase ConversorTemperatura ---

# --- Pruebas de uso para ConversorTemperatura ---
tc1 = ConversorTemperatura(25)
print("25 °C →", tc1.a_fahrenheit(), "°F")       # → 77.0
print("25 °C →", ConversorTemperatura.celsius_a_kelvin(25), "K")  # → 298.15
tc2 = ConversorTemperatura.desde_fahrenheit(77)
print("77 °F →", tc2.valor_c, "°C")              # → 25.0
tc3 = ConversorTemperatura(-300)
print("Temperatura inválida →", tc3.a_fahrenheit())  # → None (y mensaje de error)
```
