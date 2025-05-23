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

### Implementacion
```python
def closure(attributes: set[Attribute], functional_dependencies: set[FunctionalDependency]) -> set[Attribute]:
    result = set(attributes)
    changed = True

    while changed:
        changed = False
        for fd in functional_dependencies:
            if fd.lhs.issubset(result) and not fd.rhs.issubset(result):
                result.update(fd.rhs)
                changed = True
    return result
```

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

## Función `is_superkey`

### Supuestos

- `attributes` y `heading` son conjuntos de objetos `Attribute`, que deben ser hashables y comparables (por ejemplo, comparables por nombre).
- `functional_dependencies` es un conjunto de objetos `FunctionalDependency`, cada uno con:
  - `lhs`: conjunto de atributos en el lado izquierdo.
  - `rhs`: conjunto de atributos en el lado derecho.
- Se asume que existe una función `closure(attributes, functional_dependencies)` que calcula correctamente el cierre de los atributos.
- El resultado será `True` si el cierre de `attributes` incluye todos los atributos del `heading`, y `False` en caso contrario.

---

### Implementacion
```python
def is_superkey(attributes: set[Attribute], heading: set[Attribute], functional_dependencies: set[FunctionalDependency]) -> bool:
    """
    Verifica si un conjunto de atributos es una superclave, es decir, si su cierre contiene todo el encabezado.

    Parámetros:
    - attributes (set[Attribute]): Conjunto de atributos candidatos a superclave.
    - heading (set[Attribute]): Conjunto de todos los atributos de la relación.
    - functional_dependencies (set[FunctionalDependency]): Conjunto de dependencias funcionales.

    Resultado esperado:
    - bool: `True` si `attributes` es una superclave, `False` en caso contrario.
    """
    return closure(attributes, functional_dependencies) >= heading
```

### Ejemplos de uso

```python
from components import Attribute, FunctionalDependency
from algorithms import is_superkey

# Atributos
A = Attribute("A")
B = Attribute("B")
C = Attribute("C")

# Dependencias funcionales
fd1 = FunctionalDependency(lhs={A}, rhs={B})
fd2 = FunctionalDependency(lhs={B}, rhs={C})

# Encabezado de la relación
heading = {A, B, C}

# Conjunto candidato 1
attributes = {A}
fds = {fd1, fd2}
print(is_superkey(attributes, heading, fds))  # Esperado: True

# Conjunto candidato 2
attributes = {B}
print(is_superkey(attributes, heading, fds))  # Esperado: False

# Conjunto candidato 3
attributes = {A, B}
print(is_superkey(attributes, heading, fds))  # Esperado: True
```

## Función `is_key`

### Supuestos

- `attributes` y `heading` son conjuntos de objetos `Attribute`.
- `functional_dependencies` es un conjunto de objetos `FunctionalDependency`.
- Se utiliza la función `closure` para calcular el cierre de un conjunto de atributos.
- Se asume que ya existe la función `is_superkey`.

---

### Implementacion
```python
def is_key(attributes: set[Attribute], heading: set[Attribute], functional_dependencies: set[FunctionalDependency]) -> bool:
    """
    Verifica si un conjunto de atributos es una clave candidata.

    Una clave candidata es una superclave mínima, es decir, ningún subconjunto
    propio de 'attributes' debe ser una superclave.

    Args:
        attributes (set[Attribute]): Conjunto de atributos candidatos.
        heading (set[Attribute]): Conjunto de todos los atributos de la relación.
        functional_dependencies (set[FunctionalDependency]): Dependencias funcionales conocidas.

    Returns:
        bool: True si 'attributes' es superclave y ningún subconjunto propio lo es; False en caso contrario.
    """
    if not is_superkey(attributes, heading, functional_dependencies):
        return False

    # Verificamos si existe un subconjunto propio que también sea superclave
    for attr in attributes:
        subset = attributes - {attr}
        if is_superkey(subset, heading, functional_dependencies):
            return False

    return True
```

### Ejemplos de uso

```python
from components import Attribute, FunctionalDependency
from algorithms import is_key

A = Attribute("A")
B = Attribute("B")
C = Attribute("C")

fd1 = FunctionalDependency(lhs={A}, rhs={B})
fd2 = FunctionalDependency(lhs={B}, rhs={C})

heading = {A, B, C}

# A determina a B y B a C, por lo tanto A determina todo => superclave
# A no tiene subconjuntos propios, por lo tanto también es clave candidata
print(is_key({A}, heading, {fd1, fd2}))  # Esperado: True

# A y B también determinan todo, pero A sola ya lo hace, así que no es clave mínima
print(is_key({A, B}, heading, {fd1, fd2}))  # Esperado: False
```

## Función `is_relvar_in_bcnf`

### Supuestos

- `relvar` es un objeto que representa una relación con las siguientes propiedades:
- `heading`: un conjunto de objetos `Attribute` (atributos de la relación).
- `functional_dependencies`: un conjunto de objetos `FunctionalDependency`, cada uno con `lhs` y `rhs`.
- Las dependencias multivaluadas existen pero **no afectan** directamente la verificación de BCNF, ya que BCNF solo se basa en dependencias funcionales.
- Se utiliza la función `is_superkey(lhs, heading, fds)` para verificar si un conjunto de atributos es superclave.
- Solo se consideran **dependencias funcionales no triviales** (A → B, donde B no está en A).

---

### Implementacion
```python
def is_relvar_in_bcnf(relvar) -> bool:
    """
    Verifica si una relación (relvar) está en Forma Normal de Boyce-Codd (BCNF).

    Una relación está en BCNF si, para toda dependencia funcional no trivial A → B,
    A es una superclave.

    Args:
        relvar: Objeto que representa una relación, con propiedades:
            - heading: conjunto de atributos.
            - functional_dependencies: conjunto de dependencias funcionales.

    Returns:
        bool: True si la relación está en BCNF, False en caso contrario.
    """
    heading = relvar.heading
    fds = relvar.functional_dependencies

    for fd in fds:
        if not fd.rhs.issubset(fd.lhs):  # Es no trivial
            if not is_superkey(fd.lhs, heading, fds):
                return False
    return True
```

### Ejemplos de uso

```python
from components import Attribute, FunctionalDependency, Relvar
from algorithms import is_relvar_in_bcnf

A = Attribute("A")
B = Attribute("B")
C = Attribute("C")

fd1 = FunctionalDependency(lhs={A}, rhs={B})
fd2 = FunctionalDependency(lhs={B}, rhs={C})

heading = {A, B, C}
fds = {fd1, fd2}
relvar = Relvar(heading=heading, functional_dependencies=fds, multivalued_dependencies=set())

# A no es superclave y A → B existe => no está en BCNF
print(is_relvar_in_bcnf(relvar))  # Esperado: False

# Relación simple sin violaciones a BCNF
relvar2 = Relvar(
    heading={A, B},
    functional_dependencies={FunctionalDependency(lhs={A}, rhs={B})}
)
print(is_relvar_in_bcnf(relvar2))  # Esperado: True
```

## Función `is_relvar_in_4nf`

### Supuestos

- `relvar` es un objeto que representa una relación con:
- `heading`: conjunto de objetos `Attribute`.
- `functional_dependencies`: conjunto de objetos `FunctionalDependency`.
- `multivalued_dependencies`: conjunto de objetos `MultivaluedDependency`.
- `FunctionalDependency` y `MultivaluedDependency` tienen:
- `lhs`: conjunto de atributos en el lado izquierdo.
- `rhs`: conjunto de atributos en el lado derecho.
- La función `is_superkey(X, heading, fds)` determina si `X` es superclave **con base en dependencias funcionales**.
- Se consideran **dependencias no triviales**, es decir:
- Para FD: \( X \to Y \) donde \( Y \nsubseteq X \)
- Para MVD: \( X \twoheadrightarrow Y \) donde \( Y \nsubsete

---

### Implementacion
```python
def is_relvar_in_4nf(relvar) -> bool:
    """
    Verifica si una relación (relvar) está en Cuarta Forma Normal (4NF).

    Una relación está en 4NF si, para toda dependencia funcional o multivaluada no trivial X →→ Y,
    X es una superclave.

    Args:
        relvar: Objeto que representa una relación, con propiedades:
            - heading: conjunto de atributos.
            - functional_dependencies: conjunto de dependencias funcionales.
            - multivalued_dependencies: conjunto de dependencias multivaluadas.

    Returns:
        bool: True si la relación está en 4NF, False en caso contrario.
    """
    heading = relvar.heading
    fds = relvar.functional_dependencies
    mvds = relvar.multivalued_dependencies

    # Verificamos todas las dependencias funcionales no triviales
    for fd in fds:
        if not fd.rhs.issubset(fd.lhs):  # Dependencia funcional no trivial
            if not is_superkey(fd.lhs, heading, fds):
                return False

    # Verificamos todas las dependencias multivaluadas no triviales
    for mvd in mvds:
        if not mvd.rhs.issubset(mvd.lhs) and (mvd.lhs | mvd.rhs) != heading:
            if not is_superkey(mvd.lhs, heading, fds):
                return False

    return True
```

### Ejemplos de uso

```python
from components import Attribute, FunctionalDependency, MultivaluedDependency, Relvar
from algorithms import is_relvar_in_4nf

A = Attribute("A")
B = Attribute("B")
C = Attribute("C")

fd = FunctionalDependency(lhs={A}, rhs={B})
mvd = MultivaluedDependency(lhs={A}, rhs={C})

heading = {A, B, C}

relvar = Relvar(
    heading=heading,
    functional_dependencies={fd},
    multivalued_dependencies={mvd}
)

# A no es superclave, pero determina a B y multivaluadamente a C → no está en 4NF
print(is_relvar_in_4nf(relvar))  # Esperado: False

# Si A fuera superclave o no existieran dependencias multivaluadas → estaría en 4NF
```






