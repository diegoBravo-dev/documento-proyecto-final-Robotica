---
layout: default
title: Control Cinemático
nav_order: 5
---

# Control Cinemático

En esta sección, se demostrará el uso de control cinemático para que un UR30, ajeno a la solución del problema industrial, pueda dibujar dos círculos en dos puntos diferentes en el espacio.

Contenido:
- [Explicación](#explicación)
- [Ecuaciones Paramétricas](#ecuaciones-paramétricas)
- [Implementación](#implementación)
- [Demostración](#demostración)

---

## Explicación

Se trata de una técnica con el mismo propósito de la cinemática inversa, satisfacer una Pose deseada. Para ello, se deben de calcular posiciones y velocidades angulares que puedan cumplir una trayectoria deseada. 

Esta es la fórmula empleada para control Cinemático.

```python
dq = Jinv @ (dXd - K@(X - Xd))
```

Donde:

- **dq** son las velocidades angulares de cada articulación
- **Jinv** es la matriz Jacobiana inversa
- **dXd** son los perfiles de velocidad en cada dimensión de la pose deseada
- **Xd** es la pose deseada
- **X** es la pose obtenida
- **K** es la matriz de ganancias para el control cinemático

Con esta fórmula, se puede obtener un perfil de velocidades convincente, pero el parámetro deseado no son estos, sino sus integrales.

## Ecuaciones Paramétricas

Dentro de la función de Control Cinemático deben de existir las ecuaciones paramétricas para el trazado de trayectorias. 
Estas son ecuaciones que permiten crear formas geométricas y, para este ejemplo, se usará la ecuación paramétrica del círculo.

```python
x, y = c1 + r*cos(ωt), c2 + r*sin(ωt)
```

Donde

- **c1 y c2** son los centros de la circunferencia en x e y
- **r** es el radio del círculo
- **ω** es la frecuencia
- **t** es el tiempo

---

## Implementación

Primero se deben de calcular la cinemática directa y obtener las ecuaciones de x, y, z con respecto a los ángulos de las articulaciones.

> Nota: se puede incluir un control cinemático para orientaciones, pero requiere de mayor capacidad de procesamiento

Posteriormente, se debe de construir el jacobiano, derivando cada función (x, y, z) con respecto a una articulación.

```python
J = matrix([
  [j11, j12, j13, j14, j15, j16],
  [j21, j22, j23, j24, j25, j26],
  [j31, j32, j33, j34, j35, j36]
])
```
Luego, se debe de calcular la pseudo-inversa del jacobiano. Esto debido a que como el control cinemático únicamente controla 3 parámetros, la matriz jacobiana no será cuadrada.

```python
# Inversa / pseudoinversa (dependiendo del robot (n) y el problema/tarea (m))
Jinv = J.getI()
```

Después, se debe de trazar una trayectoria. Para este punto, se utilizarán las ecuaciones paramétricas y el método heuristico. 

```python
# Valores deseados de posición
if t<2:
    xd = 0.6537
    yd = -0.201
    zd = 1.0326
elif t>=2 and t<5:
    xd = 0.8187
    yd = -0.201
    zd = 1.0326
elif t>=5 and t<20:
    zd = 1.1576 + 0.1*sin(t)
    yd = -0.201 + 0.1*cos(t)
    xd = 0.8187
elif t>=20 and t<=25:
    xd = -0.4
    yd = -0.201
    zd = 1.0326
elif t>=25 and t<=40:
    zd = 1.0326 + 0.1*sin(t)
    yd = -0.201 + 0.1*cos(t)
    xd = 0.2737
else:
    xd = 0.6537
    yd = -0.201
    zd = 1.0326
```

Para las velocidades deseadas, las ecuaciones paramétricas deben de ser derivadas con respecto al tiempo.

```python
# Valores deseados de velocidad
if t<2:
    dxd = 0
    dyd = 0
    dzd = 0
elif t>=2 and t<5:
    dxd = 0
    dyd = 0
    dzd = 0
elif t>=5 and t<20:
    dzd = 0.1*cos(t)
    dyd = -0.1*sin(t)
    dxd = 0
elif t>=20 and t<=25:
    dxd = 0
    dyd = 0
    dzd = 0
elif t>=25 and t<=30:
    dxd = 0
    dyd = 0
    dzd = 0
else:
    dxd = 0
    dyd = 0
    dzd = 0
```

Ahora, se debe de construir la matriz de ganancias de control para aplicarla con el control cinemático. Las ganancias pueden variar a gusto del usuario o se pueden calcular mediante técnicas de control lineal.

```python
#Ganancias de control
kx = 1
ky = 2
kz = 3

K = matrix([[kx, 0, 0],
            [0, ky, 0],
            [0, 0, kz]])
```

Y posteriormente, calcular dq con la fórmula de control cinemático

```python
dq = Jinv @ (dXd - K@(X - Xd))
```

Ahora, esta función será añadida a un método numérico para resolver ecuaciones diferenciales ordinarias. Para python, se recomienda usar el método de Euler/Euler mejorado ya que con Rugen-Kutta, el tiempo de procesamiento puede incrementarse considerablemente.

```python
q = zeros((len(ts), 6))

q[0, :] = θs

for i in range(len(ts) - 1):

    dq = f(ts[i], q[i, :])

    q[i+1, :] = q[i, :] + h * transpose(dq)

t = array(ts)
```

tras pasar por Euler, se obtendrá un vector de tiempos y una matriz de configuraciones para las articulaciones del robot, la cual serán usadas para poder mover dicho robot en los puntos necesarios para llegar a la Pose del efector final deseada.

---

## Demostración

A continuación, se mostrará un video que ejemplifica el control cinemático aplicado a un UR30.

<iframe width="560" height="315" src="https://www.youtube.com/embed/J_IBu4eSin4?si=7oU1DSKTjamPcCrn" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Como se puede ver en el video, el robot cumple con la trayectoria impuesta y logra trazar los dos círculos en dos puntos diferentes en un tiempo establecido. Sin embargo, a diferencia de la cinemática inversa, este no es tan preciso y debido a la restricción de controlar la posición cartesiana, la orientación angular puede verse alterada de forma repentina en cada movimiento.

> Para mayor información visitar el repositorio de [Github asociado al proyecto](https://github.com/diegoBravo-dev/Proyecto-Final-de-Rob-tica-con-un-UR30-en-Robodk) en los método `PosicionarRobot()`, `f()`, `euler()`  de `UR30_Class.py` y el archivo `ControlCinematico.py` 

---

## Siguiente sección

[Resultados y Conclusiones](05-conclusion.md)