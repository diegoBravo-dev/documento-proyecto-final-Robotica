---
layout: default
title: Introducción
nav_order: 1
---

# Proyecto Pintura de una pieza con un UR30 en RoboDK 

## Diego Bravo Pérez

En esta página, se compila y documenta el trabajo desarrollado para la realización de una simulación de un proceso de pintado industrial en **RoboDK**. El proceso a recrear es ciclo completo de pintado, donde el **UR30** estará equipado con un **Generic-Paint-Sprayer** y deberá de cubrir las áreas deseadas del chasis de un **Ehang 184**.

Para poder cumplir con el ciclo, se implementaron diversas técnicas de planeación de trayectoria y obtención de una pose deseada integrando un script de **Python** con la API de **RoboDK** y los paquetes de **NumPy** y **SymPy** para las operaciones matemáticas respectivas.

- Primero, se mostrarán los elementos del espacio de trabajo necesarios para poder realizar la simulación. Esto incluye el modelo del UR30 implementado, su herramienta, una superficie donde estará el robot y por último la pieza a pintar.

- Después, se realizará una simulación de la obtención de la cinemática directa del brazo con el Generic-Paint-Sprayer. Esta etapa es muy importante ya que es la base para implementar las demás.

- Luego, se realizará la simulación del ciclo de pintado de la pieza
    - Se demostrará el cálculo de la cinemática inversa para poder obtener la configuración
      deseada que debe de tener las articulaciones del **UR30**
    - También, se demostrará los métodos empleados para la planificación de trayectoria que permitirá trazar una serie de configuraciones para llegar desde una pose inicial a una final
    - Debido a las limitaciones y algunos **bugs** del macro asociado a la herramienta empleada, no se podrá observar el efecto de pintado que **RoboDK** suele enseñar en sus simulaciones

- Por último, se hará una demostración de control cinemático fuera del problema industrial. Se trazará una trayectoria y mediante ecuaciones paramétricas, el UR30 realizará dos círculos en puntos distintos. 


Contenido:
- [1. Elementos destacables del espacio de trabajo](01-publicar-en-github-pages.md)
- [2. Cinemática Directa](02-estructura-del-repo.md)
- [3. Cinemática Inversa y Planificación de Trayectorias](03-markdown.md)
- [4. Control Cinemático](04-estilos.md)