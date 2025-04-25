# Tarea-3: Normalizacion

## FunctionalDependency - Método `is_trivial`

### Supuestos

Una **dependencia funcional** `{A1, A2, ..., An} -> {B1, B2, ..., Bm}` se considera **trivial** si **todos los atributos del lado derecho ** están incluidos en el lado izquierdo. Es decir, la dependencia es trivial si:
{B1, ..., Bm} ⊆ {A1, ..., An}


### Ejemplos de uso

```python
from components import FunctionalDependency

# Ejemplo 1: Dependencia trivial
fd1 = FunctionalDependency("{A,B} -> {A}")
print(fd1.is_trivial())  # Resultado esperado: True

# Ejemplo 2: Dependencia no trivial
fd2 = FunctionalDependency("{A,B} -> {C}")
print(fd2.is_trivial())  # Resultado esperado: False

# Ejemplo 3: Dependencia completamente trivial
fd3 = FunctionalDependency("{A,B,C} -> {A,B}")
print(fd3.is_trivial())  # Resultado esperado: True

# Ejemplo 4: Dependencia idéntica
fd4 = FunctionalDependency("{A,B} -> {A,B}")
print(fd4.is_trivial())  # Resultado esperado: True
```
## MultivaluedDependency - Método `is_trivial`

### Supuestos

Una **dependencia multivaluada** `{X} ->-> {Y}` se considera **trivial** si cumple alguma de las siguientes condiciones: 
1. `Y ⊆ X` (los atributos del lado derecho están contenidos en el izquierdo)
2. `X ∪ Y` (la union de ambos lacos cubre todo el esquema de la relaciòn)


### Ejemplos de uso

```python
from components import MultivaluedDependency, Attribute

# Definimos el esquema de la relación
heading = {Attribute("A"), Attribute("B"), Attribute("C")}

# Ejemplo 1: Y ⊆ X → trivial
mvd1 = MultivaluedDependency("{A,B} ->-> {A}")
print(mvd1.is_trivial(heading))  # Resultado esperado: True

# Ejemplo 2: X ∪ Y = heading → trivial
mvd2 = MultivaluedDependency("{A} ->-> {B,C}")
print(mvd2.is_trivial(heading))  # Resultado esperado: True

# Ejemplo 3: Ninguna de las anteriores → no trivial
mvd3 = MultivaluedDependency("{A} ->-> {B}")
print(mvd3.is_trivial({Attribute("A"), Attribute("B"), Attribute("C"), Attribute("D")}))  # Resultado esperado: False
```

## Función `closure`

### Supuestos

- Se trabaja con objetos `Attribute` como elementos del conjunto de atributos.
- Las dependencias funcionales son objetos del tipo `FunctionalDependency`.
- Cada objeto `FunctionalDependency` tiene:
  - `.determinant`: conjunto de atributos en el lado izquierdo (X).
  - `.dependant`: conjunto de atributos en el lado derecho (Y).
- Se asume que todos los atributos involucrados en las dependencias pertenecen al esquema relacional.

---

### Ejemplos de uso

```python
from components import Attribute, FunctionalDependency
from algorithms import closure

# Atributos
A = Attribute("A")
B = Attribute("B")
C = Attribute("C")
D = Attribute("D")

# Conjunto inicial de atributos
attrs = {A}

# Conjunto de dependencias funcionales
fds = [
    FunctionalDependency("{A} -> {B}"),
    FunctionalDependency("{B} -> {C}"),
    FunctionalDependency("{C} -> {D}")
]
```

### Calcular cierre
```python
result = closure(attrs, fds)
print({attr.name for attr in result})  # Resultado esperado: {'A', 'B', 'C', 'D'}
```






