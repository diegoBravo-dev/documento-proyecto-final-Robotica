---
layout: default
title: Elementos destacables del espacio de trabajo
nav_order: 2
---

# Espacio de trabajo y elementos que lo conforman

En esta página, se muestran y detallan los elementos utilizados para la simulación en **RoboDK**

Contenido:
- [UR30](#brazo-de-universal-robots-ur30)
- [2. Cinemática Directa](02-estructura-del-repo.md)
- [3. Cinemática Inversa y Planificación de Trayectorias](03-markdown.md)
- [4. Control Cinemático](04-estilos.md)

---

## Brazo de Universal Robots UR30

El robot UR30 es un brazo robótico de 6 ejes, tiene una carga útil de 30.0 kg y un alcance de 1300 mm. Además, puede levantar cargas pesadas manteniendo un tamaño compacto en un entorno colaborativo.

Sus aplicaciones más comunes en la industria son: 

   - Ideal para la alimentación de máquinas
   - Manipulación de materiales 
   - Paletizado de productos pesados 
   - Atornillado de alto par.


![Brazo colaborativo UR30](assets/img/ur30.png)

> Modelo del brazo colaborativo [UR30](https://robodk.com/3D/es/robot/UR30) extraído desde **RoboDK**

### Características principales del UR30

   - **Seis grados de libertad** representado en sus seis articulaciones que lo componen
   - **Repetitividad de ± 0.1 mm** que garantiza una precisión constantes en tareas que se repiten con frecuencia
   - **Peso de 65 kg** lo que lo hace relativamente ligero y fácil de reubicar
   - **Consumo Eléctrico de aproximadamente 300 W** en uso típico (máximo de 750 W).
   - **Torque elevado** para manejar un par de torsión alto
   - **Diversas funciones de seguridad configurables**, como detección de fuerza y colisión, permitiéndole trabajar junto a operarios sin necesidad de vallas de seguridad
   - **Grado de Protección con certificación IP65**, lo que lo protege contra el polvo y chorros de agua
   - **Huella Compacta con una base de solo Ø 245 mm** para instalarse en celdas de trabajo donde el espacio es muy limitado.

### Parámetros de Denavit-Hatenberg 

Los parámetros de Denavit-Hartenberg (DH) son un método sistemático para representar las cadenas cinemáticas de los brazos robóticos. Simplifican el modelado matemático de robots al proporcionar una notción estándar para describir las posiciones y orientaciones relativas de los enlaces adyacentes. Los cuatro parámetros DH —longitud del enlace, torsión del enlace, desplazamiento del enlace y ángulo de la articulación— permiten la descripción precisa de cada articulación en términos de un sistema de coordenadas común, lo que facilita la derivación de las ecuaciones cinemáticas necesarias para controlar el movimiento del robot.

A continuación se mostrará un diagrama que ayuda a la forma en la que se obtienen dichos parámetros.

![Diagrama de parámetros DH](assets/img/kinematic.png)

> Diagrama de parámetros de DH para robots [UR](https://www.universal-robots.com/articles/ur/application-installation/dh-parameters-for-calculations-of-kinematics-and-dynamics/). Extraído desde la página oficial de Universal Robots


Los parámetros d, alpha y a se mostrarán en la siguiente tabla:

| Junta     | a         | d            | alpha       |
|----------:|:---------:|:-------------|:------------|
| Junta 1   | 0         | 0.2363       | π/2         |
| Junta 2   | -0.6370   | 0            | 0           |
| Junta 3   | -0.5037   | 0            | 0           |
| Junta 4   | 0         | 0.2010       | π/2         |
| Junta 5   | 0         | 0.1593       | -π/2        |
| Junta 6   | 0         | 0.1543       | 0           |


> Parámetros de DH para robots [UR](https://www.universal-robots.com/articles/ur/application-installation/dh-parameters-for-calculations-of-kinematics-and-dynamics/). Extraído desde la página oficial de Universal Robots


---

## Siguiente sección
[Estructura del repositorio ](02-estructura-del-repo.md)