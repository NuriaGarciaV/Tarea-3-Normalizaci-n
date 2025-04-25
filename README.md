# Tarea-3-Normalizacion

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



