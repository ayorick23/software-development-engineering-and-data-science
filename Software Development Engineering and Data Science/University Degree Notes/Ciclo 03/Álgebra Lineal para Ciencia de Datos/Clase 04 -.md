---
Fecha de creación: 2026-02-22T14:59:00
Materia:
  - Álgebra Lineal para Ciencia de Datos
Fecha de clase: 2026-02-14
---
Matriz inversa
Matriz adjunta
Ley de sarrus
Cofactores
Método Gauss de Jordan

Sistemas lineales
Regla de Crammer

Ejercicios ejemplo con Matriz Adjunta en python y su homologo en matlab
Ejercicio ejemplo con sistema de ecuaciones

```python
import numpy as np

#Ax=b
A=np.array([[2,3],[4,5]],dtype=float)
b=np.array([8,14], dtype=float)

print("sistema de ecuaciones lineales")
print("2x + 3y = 8")
print("4x + 5y = 14")
print("\nMatriz A")
print(A)
print("\nVector b")
print(b)


# Paso 1: Calcular el determinante
det=np.linalg.det(A)
print(f"\nDeterminante de A: {det:.2f}")

#Paso 2: Calcular matriz de cofactores
cofactores=np.array([[A[1,1], -A[1,0]], [-A[0,1], A[0,0]]])
print("\nMatriz de cofactores")
print(cofactores)

#Paso 3: Matriz adjunta(transpuesta de cofactores)
adjunta=cofactores.T
print("\nMatriz adjunta")
print(adjunta)

#Paso 4: Calcular la inversa: A^-1=adj(A)/det(A)
inversa=adjunta/det
print("\nMatriz inversa")
print(inversa) 

#verificar la inversa en python
verificar=np.linalg.inv(A)
print("\nVerificar inversa")
print(verificar)

#resolver el sistema de ecuaciones lineales
reso=adjunta@b
x1=np.linalg.solve(A,b)

print("\nSolución del sistema de ecuaciones")
print(reso)
print(f"Usando adjunta: x={reso[0]:.2f}, y={reso[1]:.2f}")
print(f"Usando solve: x={x1[0]:.2f}, y={x1[1]: .2f}")
```

