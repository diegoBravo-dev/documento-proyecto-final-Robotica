---
layout: default
title: Cinemática Inversa y Planificación de trayectoria
nav_order: 4
---

# Cinemática Inversa y Planificación de trayectoria

En esta sección, se abordará el método empleado para la cinemática inversa y los métodos de planificación de trayectoria para poder resolver el problema industrial.

Contenido:
- [Cinemática Inversa](#cinemática-inversa)
  - [Método](#método-de-levenberg-marquadt)
- [Planificación de trayectorias](#planificación-de-trayectoria-mediante-polinomio-quíntico)
  - [Método de Polinomio Quíntico](#método-de-polinomio-quíntico)
  - [Método Heurístico](#método-heurístico)
- [Demostración](#demostración)

---

## Cinemática Inversa

La cinemática inversa consiste en satisfacer una pose deseada, es decir una posición y orientación deseada para el Efector Final. Por lo tanto, lo que se busca obtener de esta operación es una configuración adecuada de las articulaciones que lleven la herramienta a un punto en el espacio de referencia deseado.

Para ello, existen diversos métodos que permiten resolver este problema. El que se empleó en este proyecto fue un método numérico.

### Método de Levenberg-Marquadt

Es un algoritmo de optimización que resuelve problemas de mínimos cuadrados no lineales. Combina lo mejor del método de Gauss-Newton y Gradiantes descendentes, transicionando suavemente entre ambos según qué tan cerca está del mínimo.

En robótica es un método comunmente usado para robots quirurgicos, brazos manipuladores, prótesis y androides.

Para poder implementarlo, se deben de dar una pose deseada como argumento, es decir, una posición cartesiana (en m) y una orientación angular (en éje-ángulo).

Primero, se debe de inicializar los parámetros de optimización para realizar la evaluación numérica:

```python
# PARÁMETROS DE OPTIMIZACIÓN
max_iteraciones = 1000
tol = 1e-8
mu = 1e-3
q = array(deg2rad(θ))
```

> Nota: Siempre se debe de trabajar en radianes para realizar este tipo de operaciones

Posteriormente, se debe de calcular la **Cinemática Directa** del Efector Final con respecto a la referencia, además de guardar cada matriz homogenea calculada durante la iteración de la Cinemática directa. Esta será útil para obtener el **Jacobiano**

```python
T0_tool, T_stack = self.DH_Solve_t_stack(qs)

posicionTool = [T0_tool[0][3], T0_tool[1][3], T0_tool[2][3]]
rotacionTool = T0_tool[:3, :3]
```

También, se debe de transformar el vector éje-ángulo a su equivalente en **Matriz de rotación**

Para obtener la matriz de rotación, se debe de aplicar la fórmula de Rodrigues, la cual necesita de diversos elementos. El primero es ángulo de rotación, que se obtiene con la normal del vector eje-ángulo.

```python
theta = norm(rv)
```

El segundo, es obtener el eje unitario, que se obtiene dividiendo el vector éje-ángulo sobre el ángulo de rotación.

```python
u = rv / theta
```

Posteriormente, hay que obtener la matriz antisimétrica del eje unitario.

```python
Ux = array([[0, -u[2], u[1]],
            [u[2], 0, -u[0]],
            [-u[1], u[0], 0]])
```

Y finalmente, aplicar la fórmula de Rodrigues para obtener la matriz de orientación

```python
R = eye(3) + sin(theta) * Ux + (1 - cos(theta)) * (Ux @ Ux)
```

Posteriormente, se debe de construir el jacobiano geométrico, columna por columna.

```python
o_n = T0_tool[:3, 3]          # posición del TCP en mundo
z_p = array([0, 0, 1])        # eje z de la base (frame 0)
o_p = array([0, 0, 0])        # origen de la base

J[:, 0] = concatenate([cross(z_p, o_n - o_p), z_p])

for i in range(1, 6):
    z_i = T_stack[i-1][:3, 2]   # eje z del frame i
    o_i = T_stack[i-1][:3, 3]   # origen del frame i
    J[:, i] = concatenate([cross(z_i, o_n - o_i), z_i])
```

Además, se debe de obtener los errores de orientación y posición, para aquel se debe de aplicar un producto cruz por las dos matrices de rotación.

```python
#Error de posición
ep = array([xd - posicionTool[0],
            yd - posicionTool[1],
            zd - posicionTool[2]])
        
#Error de Orientación (Vectorial para compatibilidad con J)
ew_vec = 0.5 * (cross(rotacionTool[:,0], Rd[:,0]) + cross(rotacionTool[:,1], Rd[:,1]) + cross(rotacionTool[:,2], Rd[:,2]))
```

Y posteriormente, realizar una comprobación en caso de que exista una singularidad:

```python
if det(J) == 0:
  error_f = norm(ep) + norm(ew_vec) + 1000
  print("--- Singularidad ---\n")
else:
  error_f = norm(ep) + norm(ew_vec)

```

Una vez con estos datos obtenidos, se puede realizar una comprobación inicial. En caso de que el error final es menor a la toleracia, se rompe el ciclo de iteraciones

Posteriormente, se realiza el cálculo de Levenberg-Marquadt:

```python
#Cálculo de Levenberg-Marquadt
A = J.T @ J + mu * eye(6)
g = J.T @ e_vector
dq = solve(A, g)

q_new = q + dq
```

Se evalua la mejora, y si el error nuevo es menor al actual, entonces la nueva configuración de las articulaciones se guarda y el parámetro de mu (que dictamina si confiar más en Guass-Newton o en Gradiantes descendentes) se multiplica o divide por 10

```python
if error_nuevo < error_actual and not any(isnan(q_new)):
    q = q_new
    mu /= 10
else:
    mu *= 10
```

> Nota: importante transformar la configuración final a grados, pues RoboDK y Universal Robots trabajan con estos.

---

## Planificación de trayectorias

Una vez obtenido la pose deseada, se debe de trazar un listado de configuraciones que se deben de modificar en tiempo real para poder llegar de una pose A a una B. A esto se le conoce como planificación de trayectoria.

Al igual que la cinemática inversa, existen muchos métodos que permiten la creación de dichas trayectorias. Sin embargo, sólo se van a abordar dos.

### Método de polinomio quíntico

Una trayectoria quíntica se define mediante un polinomio de quinto grado, lo que significa que tiene seis coeficientes que determinan completamente el movimiento.

La principal ventaja del polinomio quíntico es que permite controlar simultáneamente tres magnitudes físicas (posición, velocidad y aceleración) en los puntos inicial y final de la trayectoria, garantizando un movimiento completamente predecible.

Para poder Implementarlo, se debe de tomar en cuenta que se cuenta con 6 grados de libertad a controlar, por lo que las operaciones deberán de ser vectoriales y matriciales.

Primero, se debe de establecer los parámetros iniciales y finales de las velocidades angulares y aceleraciones angulares de cada articulación, además de establecer parámetros de tiempo para generar la trayectoria con respecto a un tiempo y paso definido previamente.

También se debe de crear la matriz de tiempos, la cuál sera importante para hallar los coeficientes indicados para obtener la configuración angular adecuada.

```python
ω0 = zeros(6)
α0 = zeros(6)
ωf = zeros(6)
αf = zeros(6)

t0 = 0
tf = 10
h = 0.5

A = array([
  [t0**5   , t0**4   , t0**3  , t0**2, t0, 1],
  [5*t0**4 , 4*t0**3 , 3*t0**2, 2*t0 , 1 , 0],
  [20*t0**3, 12*t0**2, 6*t0   , 2    , 0 , 0],
  [tf**5   , tf**4   , tf**3  , tf**2, tf, 1],
  [5*tf**4 , 4*tf**3 , 3*tf**2, 2*tf , 1 , 0],
  [20*tf**3, 12*tf**2, 6*tf   , 2    , 0 , 0]
])
    
t = arange(t0, tf+h, h)

θ_all = []
ω_all = []
α_all = []
```

Ahora, se debe de construir la matriz de requisitos de movimiento, la cual se evaluará con los valores de la matriz de tiempos y su resultado darán los coeficientes para el polinomio quíntico.

```python
B = array([
    θ0[i], ω0[i], α0[i],
    θf[i], ωf[i], αf[i],
])

X = solve(A, B)
a5, a4, a3, a2, a1, a0_ = X
```

Con el polinomio quíntico, se debe de obtener las configuraciones para la posición angular, la aceleración angular y velocidad angular.

```python
q = a5*t**5 + a4*t**4 + a3*t**3 + a2*t**2 + a1*t + a0_
w = 5*a5*t**4 + 4*a4*t**3 + 3*a3*t**2 + 2*a2*t + a1
acc = 20*a5*t**3 + 12*a4*t**2 + 6*a3*t + 2*a2

θ_all.append(q)
ω_all.append(w)
α_all.append(acc)
```

Por último, estos pasos se tienen que iterar de acuerdo al número de articulaciones del robot. Para este caso, sólo basta con seis veces.

### Método heurístico

Para el método herístico únicamente basta con trazar una trayectoria que vaya cambiando la posición angular con respecto al tiempo en el que se encuentre.

Es decir, se desea llegar de un punto A a un punto D, pasando por B y C. Lo que hace el método heurístico es que se dependiendo el instante del tiempo, el punto se moverá a A, B o C hasta llegar a D en un determinado momento.

---

## Demostración

A continuación, se mostrará un video que implementa estas tres técnicas para trazar la trayectoria y realizar una parte de la tarea completa del robot.

<video controls width="720">
  <source src="/workspaces/documento-proyecto-final-Robotica/assets/videos/ci_pt.mp4" type="video/mp4">
  Demostración de la cinemática directa combinada con los métodos de planificación de trayectoria
</video>


Como se puede ver en el video, primero se utiliza Levenberg-Marquadt para calcular la configuración que debe de tener cada articulación del robot para llegar a una pose deseada desde una pose definida (esta se actualiza después de cada trayectoria). Posteriormente se calcula la trayectoria y el robot se desplaza por esta misma hasta llegar a la pose que LM calculó previamente.


> Para mayor información visitar el repositorio de [Github asociado al proyecto](https://github.com/diegoBravo-dev/Proyecto-Final-de-Rob-tica-con-un-UR30-en-Robodk) en los método `CinematicaInversa(self)` de `UR30_Class.py` y el archivo `trayectorias.py` 

---

## Siguiente sección

[Control Cinemático](04-control-cin.md)