## Escenario Procedural en Blender 
Este programa está hecho en Python y se ejecuta dentro de Blender para crear automáticamente un escenario en 3D, el código construye un pasillo con paredes de colores alternados, un suelo con forma curva y una cámara que se mueve a lo largo del recorrido.
Primero, el programa limpia la escena para empezar desde cero después crea los materiales que se usarán para dar color a las paredes y al suelo. Luego se definen algunos valores que controlan el tamaño del pasillo, cuándo empieza la curva y qué tan pronunciada será. Con ayuda de cálculos matemáticos, el programa genera las posiciones necesarias para formar un pasillo que comienza recto y luego se curva suavemente a partir de esas posiciones se crean las paredes usando cubos y el suelo usando una malla personalizada por ultimo se agrega una cámara que se anima automáticamente para avanzar por el pasillo.
## Importacion de modulos 
Al inicio del programa se importan tres módulos que permiten que el script funcione correctamente dentro de Blender. Estos módulos le dan al código acceso a herramientas específicas necesarias para crear objetos, realizar cálculos matemáticos y construir geometría personalizada.

**bpy:** Es el módulo que permite controlar Blender desde Python. Es fundamental porque sin él no se pueden crear ni modificar objetos en la escena.

**math:** Es una librería matemática de Python que permite usar funciones como seno y conversión a radianes. Es importante para realizar cálculos que generan movimiento y curvas.

**bmesh:** Es un módulo que permite crear y modificar mallas de manera manual y detallada. Es importante para construir geometría personalizada como el suelo curvo del pasillo.

```python
import bpy
import math
import bmesh
```

## Preparacion del entorno de trabajo

En esta sección del código se eliminan todos los objetos existentes en la escena de Blender antes de comenzar a crear nuevos elementos. Esto es importante para evitar que queden objetos anteriores que puedan interferir con el resultado final del proyecto. Primero se utiliza la instrucción bpy.ops.object.select_all(action='SELECT'), la cual selecciona todos los objetos presentes en la escena actual. Es equivalente a presionar la tecla A en Blender para seleccionar todo. Después se ejecuta bpy.ops.object.delete(), que elimina todos los objetos previamente seleccionados. Como en el paso anterior se seleccionaron todos, esta instrucción borra completamente la escena. Esta sección es fundamental porque garantiza que el script trabaje en un entorno limpio y controlado, evitando errores, duplicaciones o mezclas con elementos creados anteriormente.

**bpy.ops.object.select_all(action=‘SELECT’):** Selecciona todos los objetos de la escena.

**bpy.ops.object.delete():** Elimina todos los objetos seleccionados.

```python
#  LIMPIAR ESCENA 
bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete()
```
##  Creacion de Materiales 
En esta sección del código se definen los materiales que se utilizarán para dar color a los objetos del escenario. En Blender, los materiales permiten modificar la apariencia visual de los objetos, como su color, brillo y textura. Primero se crea una función llamada crear_material. Esta función recibe dos parámetros: el nombre del material y un color en formato RGB (rojo, verde y azul). Dentro de la función se genera un nuevo material usando bpy.data.materials.new(), asignándole el nombre proporcionado. Después, se establece el color del material mediante la propiedad diffuse_color. Aquí se utiliza la sintaxis (*color_rgb, 1.0), donde los tres primeros valores corresponden al color RGB y el 1.0 representa el nivel de opacidad (alpha), indicando que el material es completamente visible.

Finalmente, la función retorna el material creado para poder usarlo posteriormente en los objetos.

Luego de definir la función, se crean tres materiales específicos:
Un material negro para las paredes.

-Un material morado para alternar el color de los bloques.

-Un material gris para el suelo.

Esto permite reutilizar materiales de manera organizada y evitar repetir código.

**def crear_material(nombre, color_rgb):** Define una función para crear materiales personalizados.

**bpy.data.materials.new(name=nombre):** Crea un nuevo material en Blender.

**mat.diffuse_color:** Asigna el color del material en formato RGBA.

**return mat:** Devuelve el material para poder usarlo después.

**mat_negro, mat_morado, mat_suelo:** Materiales creados a partir de la función definida.

```python
#  MATERIALES 
def crear_material(nombre, color_rgb):
    mat = bpy.data.materials.new(name=nombre)
    mat.diffuse_color = (*color_rgb, 1.0)
    return mat

mat_negro = crear_material("Negro", (0.05, 0.05, 0.05))
mat_morado = crear_material("Morado", (0.5, 0.0, 0.8))  # Morado
mat_suelo = crear_material("SueloGris", (0.4, 0.4, 0.4))

```

## Definicion de Parametros 

En esta sección se establecen las variables que controlan la forma y dimensiones del pasillo. Estos valores funcionan como configuraciones generales que determinan cómo se construirá la estructura dentro de la escena. Primero se define largo_pasillo, que indica la cantidad de segmentos que tendrá el pasillo. Este valor influye directamente en qué tan largo será el recorrido. Después se define ancho_pasillo, que determina la distancia desde el centro hacia cada lado donde se colocarán las paredes. Es decir, controla qué tan ancho será el espacio por donde se moverá la cámara. La variable inicio_curva indica en qué punto del pasillo comenzará la curvatura. Antes de ese número, el pasillo se mantiene recto, la variable amplitud controla qué tan pronunciada será la curva. Un valor mayor generará una curva más amplia.Luego, longitud_segmento define cuánto avanza el pasillo en cada iteración del bucle. Es decir, establece la distancia entre cada sección del recorrido, por último, grosor_pared ajusta la separación interna del suelo respecto a las paredes, evitando que el suelo se sobreponga con ellas , esta sección es importante porque permite modificar fácilmente el diseño del pasillo sin cambiar la lógica principal del código.

**largo_pasillo:** Número total de segmentos del pasillo.

**ancho_pasillo:** Distancia desde el centro hasta cada pared.

**inicio_curva:** Punto donde comienza la curvatura.

**amplitud:** Intensidad de la curva.

**longitud_segmento:** Distancia entre cada segmento del pasillo.

**grosor_pared:** Ajuste de separación entre suelo y paredes.

```python
# ---------------- PARÁMETROS ----------------
largo_pasillo = 50
ancho_pasillo = 3
inicio_curva = 15
amplitud = 6
longitud_segmento = 2
grosor_pared = 0.8
```

## Calculo de Posiciones del Pasillo 

En esta sección del código se calculan las coordenadas que determinarán la forma del pasillo. Básicamente, aquí se decide en qué posición estará cada segmento a lo largo del recorrido, incluyendo la parte curva.Primero se crea una lista llamada posiciones, que inicialmente contiene una tupla con los valores (0, 0, 0). Estos valores representan la posición inicial del pasillo en el espacio tridimensional (x, y, z o en este caso dirección), después se inicializan tres variables: **x_actual**,**y_actual** y **direccion_actual**, todas con valor cero, estas variables guardarán la posición actual mientras se va construyendo el pasillo paso a paso. Luego comienza un ciclo for que se ejecuta tantas veces como indique largo_pasillo. En cada repetición se calcula una nueva posición, la variable n se utiliza para determinar cuándo debe empezar la curva. Con max(0, i - inicio_curva) se evita que el valor sea negativo. Esto significa que antes del punto definido en inicio_curva, la curva no se aplica.

Después se calcula offset, que es el desplazamiento horizontal en el eje X. Este desplazamiento se obtiene usando la función seno (math.sin), lo que permite generar una curva suave. La fórmula también incluye la amplitud para controlar qué tan pronunciada es la curva y un factor progresivo min(1.0, n/10) para que la curva no aparezca de forma brusca, sino que aumente gradualmente luego se actualiza x_actual con el valor del desplazamiento calculado, la variable y_actual se incrementa en cada iteración usando longitud_segmento, lo que hace que el pasillo avance hacia adelante.Finalmente, se agrega la nueva posición calculada a la lista posiciones mediante append. De esta manera se va almacenando cada punto que luego servirá para construir las paredes y el suelo curvo.

Esta sección es fundamental porque define matemáticamente la forma del recorrido.

**posiciones:** Lista que almacena las coordenadas del pasillo.

**for i in range(largo_pasillo):** Bucle que genera cada segmento.

**n = max(0, i - inicio_curva):** Controla cuándo inicia la curva.

**math.sin():** Genera el movimiento curvo.

**offset:** Desplazamiento horizontal en el eje X.

**y_actual += longitud_segmento:** Hace avanzar el pasillo.

**append():** Guarda cada nueva posición en la lista.

```python
#  CALCULAR POSICIONES 
posiciones = [(0, 0, 0)]
x_actual, y_actual, direccion_actual = 0, 0, 0

for i in range(largo_pasillo):
    n = max(0, i - inicio_curva)
    offset = math.sin(n * 0.3) * amplitud * min(1.0, n/10)

    x_actual = offset
    y_actual += longitud_segmento
    posiciones.append((x_actual, y_actual, direccion_actual))

```
## Creacion de las paredes del pasillo 

En esta sección del código se generan las paredes del pasillo utilizando cubos. Estos cubos se colocan a ambos lados del recorrido siguiendo las posiciones previamente calculadas.Primero se inicia un ciclo for que recorre la lista posiciones. Se utiliza len(posiciones) - 1 porque se trabaja comparando segmentos consecutivos y se evita salir del rango de la lista, en cada iteración se extraen los valores x, y y angulo desde la lista posiciones. Estas coordenadas indican dónde debe colocarse cada sección del pasillo.

Para crear la pared izquierda, se usa bpy.ops.mesh.primitive_cube_add() y se coloca el cubo en la posición (x - ancho_pasillo, y, 1). Restar ancho_pasillo en el eje X desplaza el cubo hacia la izquierda del centro del pasillo. El valor 1 en el eje Z eleva el cubo ligeramente para que quede sobre el suelo, después se guarda el cubo recién creado en la variable pared_izq usando bpy.context.active_object.

Luego se ajusta su tamaño con scale = (0.8, 0.8, 2), esto hace que el cubo sea más alto en el eje Z y más delgado en los ejes X y Y dándole forma de pared, posteriormente  se asigna el material utilizando una condición: **mat_negro if i % 2 == 0 else mat_morado**. El operador % es el módulo, que calcula el residuo de la división entre 2. Si el resultado es 0, significa que el número es par. Esto permite alternar los colores entre negro y morado para crear un patrón visual. El mismo procedimiento se repite para la pared derecha, pero en lugar de restar ancho_pasillo, se suma, esto coloca el cubo al lado contrario del pasillo. También se invierte el orden de los materiales para que el patrón alternado sea simétrico.


**for i in range(len(posiciones) - 1):** Recorre todas las posiciones calculadas.

**bpy.ops.mesh.primitive_cube_add():** Crea un cubo en la posición indicada.

**bpy.context.active_object:** Obtiene el objeto recién creado.

**scale:** Modifica el tamaño del cubo.

**i % 2:** Permite alternar colores entre negro y morado.

**ancho_pasillo:** Controla la distancia de las paredes respecto al centro.


```python
# ---------------- CREAR PAREDES (CUBOS) ----------------
for i in range(len(posiciones) - 1):
    x, y, angulo = posiciones[i]

    # Izquierda
    bpy.ops.mesh.primitive_cube_add(location=(x - ancho_pasillo, y, 1))
    pared_izq = bpy.context.active_object
    pared_izq.scale = (0.8, 0.8, 2)
    pared_izq.data.materials.append(mat_negro if i % 2 == 0 else mat_morado)

    # Derecha
    bpy.ops.mesh.primitive_cube_add(location=(x + ancho_pasillo, y, 1))
    pared_der = bpy.context.active_object
    pared_der.scale = (0.8, 0.8, 2)
    pared_der.data.materials.append(mat_morado if i % 2 == 0 else mat_negro)

```

## Creacion del suelo 
En esta sección se construye el suelo del pasillo utilizando geometría personalizada. A diferencia de las paredes, que se crean con cubos predeterminados, aquí se genera una malla manualmente usando el módulo bmesh. Primero se crean dos listas vacías llamadas borde_izq y borde_der. Estas listas almacenarán los puntos que formarán los bordes izquierdo y derecho del suelo, luego se recorre la lista posiciones. En cada iteración se agregan nuevos puntos a ambas listas.
Para el borde izquierdo se calcula la posición restando ancho_pasillo y sumando grosor_pared, mientras que para el borde derecho se suma ancho_pasillo y se resta **grosor_pared** esto permite que el suelo quede ligeramente dentro de las paredes y no se sobreponga con ellas. El valor 0 en el eje Z indica que el suelo estará al nivel base.

Después se crea una nueva malla con bpy.data.meshes.new("SueloCurvo") y luego se crea un objeto usando esa malla y se enlaza a la colección actual para que aparezca en la escena. A continuación, se crea un objeto bmesh nuevo con bmesh.new(). Este permite construir la geometría manualmente, se crean los vértices del lado izquierdo y derecho usando comprensión de listas, cada punto guardado anteriormente se convierte en un vértice real dentro de la malla.Después, mediante un ciclo for, se crean las caras del suelo. Cada cara conecta cuatro vértices consecutivos (dos del lado izquierdo y dos del lado derecho). Esto forma una serie de rectángulos que componen la superficie del suelo una vez construida la geometría, se transfiere la información del bmesh a la malla real con bm.to_mesh(mesh) y luego se libera la memoria con bm.free(), posteriormente se asigna el material gris al suelo finalmente, se añade un modificador llamado “Subdivision” que suaviza la geometría, haciendo que el suelo se vea más curvo y menos segmentado. También se aplica shade_smooth() para suavizar visualmente las superficies, esta sección es clave porque transforma los cálculos matemáticos en una superficie continua y real dentro del entorno 3D.

**borde_izq / borde_der:** Listas que almacenan los puntos del suelo.

**bpy.data.meshes.new():** Crea una nueva malla.

**bmesh.new():** Permite construir geometría manualmente.

**bm.verts.new():** Crea vértices.

**bm.faces.new()** Crea caras conectando vértices.

**bm.to_mesh():** Transfiere la geometría al objeto real.

**Subdivision Surface:** Suaviza la malla.

**shade_smooth():** Mejora el aspecto visual.

```python
#  CREAR SUELO CURVO 
borde_izq = []
borde_der = []

for x, y, angulo in posiciones:
    borde_izq.append((x - ancho_pasillo + grosor_pared, y, 0))
    borde_der.append((x + ancho_pasillo - grosor_pared, y, 0))

mesh = bpy.data.meshes.new("SueloCurvo")
obj_suelo = bpy.data.objects.new("SueloCurvo", mesh)
bpy.context.collection.objects.link(obj_suelo)

bm = bmesh.new()
verts_izq = [bm.verts.new(p) for p in borde_izq]
verts_der = [bm.verts.new(p) for p in borde_der]

for i in range(len(verts_izq) - 1):
    bm.faces.new((verts_izq[i], verts_der[i],
                  verts_der[i + 1], verts_izq[i + 1]))

bm.to_mesh(mesh)
bm.free()

obj_suelo.data.materials.append(mat_suelo)

subsurf = obj_suelo.modifiers.new(name="Subdivision", type='SUBSURF')
subsurf.levels = 2
bpy.context.view_layer.objects.active = obj_suelo
bpy.ops.object.shade_smooth()


```

## Animacion de la Camara a lo largo del pasillo 

En esta sección del código se crea una cámara y se programa su movimiento para que recorra automáticamente el pasillo generado, primero se agrega una cámara a la escena con bpy.ops.object.camera_add(). Luego se guarda la cámara recién creada en la variable cam utilizando bpy.context.active_object, lo que permite modificar sus propiedades, después se ajusta su rotación con **cam.rotation_euler = (math.radians(90), 0, 0)**, aquí se convierte 90 grados a radianes porque Blender trabaja internamente en radianes. Esta rotación orienta la cámara correctamente para que apunte hacia el recorrido del pasillo, luego se inicia un ciclo for que va del fotograma 1 al 250. Esto significa que la animación tendrá 250 cuadros (frames).

Dentro del ciclo se calcula la variable progreso, que representa cuánto ha avanzado la cámara a lo largo del pasillo. Se obtiene dividiendo el fotograma actual entre el total y multiplicándolo por la longitud total del recorrido, después se calcula n_cam, que controla el inicio de la curva para la cámara, de manera similar a como se hizo al construir el pasillo, la variable offset_cam usa la función seno para calcular el desplazamiento lateral en el eje X. Esto permite que la cámara siga exactamente la misma curva que el pasillo.

Luego se actualizan las coordenadas de la cámara:

-En el eje X se aplica el desplazamiento lateral.

-En el eje Y se mueve hacia adelante según el progreso.

-En el eje Z se mantiene a una altura fija de 1.5 unidades.

Finalmente, se utiliza **cam.keyframe_insert()** para guardar la posición de la cámara en cada fotograma. Esto crea automáticamente la animación, haciendo que la cámara se desplace suavemente a lo largo del pasillo. Esta sección es importante porque convierte el escenario estático en una experiencia dinámica, permitiendo visualizar el recorrido en movimiento.


**camera_add():** Agrega una cámara a la escena.

**rotation_euler:** Ajusta la orientación de la cámara.

**range(1, 251):** Define la duración de la animación.

**progreso:** Calcula el avance de la cámara.

**math.sin():** Permite que la cámara siga la curva.

**cam.location:** Define la posición en X, Y y Z.

**keyframe_insert():** Guarda la posición en cada fotograma para crear animación.

```python
#  CÁMARA ANIMADA 
bpy.ops.object.camera_add()
cam = bpy.context.active_object
cam.rotation_euler = (math.radians(90), 0, 0)

for f in range(1, 251):
    progreso = (f / 250) * (largo_pasillo - 1)

    n_cam = max(0, progreso - inicio_curva)
    offset_cam = math.sin(n_cam * 0.3) * amplitud * min(1.0, n_cam/10)

    cam.location.x = offset_cam
    cam.location.y = progreso * longitud_segmento
    cam.location.z = 1.5

    cam.keyframe_insert(data_path="location", frame=f)

```

##  Iluminacion de la escena 
En esta sección final del código se agrega una fuente de luz para iluminar el pasillo. La iluminación es fundamental en un entorno 3D porque permite que los objetos sean visibles y que los materiales se aprecien correctamente, se utiliza la instrucción bpy.ops.object.light_add() para añadir una luz a la escena. El parámetro type='SUN' indica que se está creando una luz tipo Sol.

La luz tipo Sol emite iluminación uniforme en una dirección específica, similar a la luz solar real. No depende de la distancia entre la luz y los objetos, lo que la hace ideal para iluminar escenas completas de manera consistente, el parámetro **location=(0, 0, 15)** establece la posición de la luz en el espacio tridimensional. En este caso, se coloca a una altura de 15 unidades en el eje Z, por encima del pasillo, lo que permite que la luz ilumine la escena desde arriba en esta ultim parte es importante porque sin iluminación los objetos podrían verse completamente oscuros, especialmente en el modo de renderizado.

**light_add():** Agrega una fuente de luz a la escena.

**type=‘SUN’:** Define que la luz será tipo Sol (iluminación uniforme).

**location=(0, 0, 15):** Posición de la luz en el espacio 3D.

## Codigo 

```python

import bpy
import math
import bmesh

# ---------------- LIMPIAR ESCENA ----------------
bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete()

# ---------------- MATERIALES ----------------
def crear_material(nombre, color_rgb):
    mat = bpy.data.materials.new(name=nombre)
    mat.diffuse_color = (*color_rgb, 1.0)
    return mat

mat_negro = crear_material("Negro", (0.05, 0.05, 0.05))
mat_morado = crear_material("Morado", (0.5, 0.0, 0.8))  # Morado
mat_suelo = crear_material("SueloGris", (0.4, 0.4, 0.4))

# ---------------- PARÁMETROS ----------------
largo_pasillo = 50
ancho_pasillo = 3
inicio_curva = 15
amplitud = 6
longitud_segmento = 2
grosor_pared = 0.8

# ---------------- CALCULAR POSICIONES ----------------
posiciones = [(0, 0, 0)]
x_actual, y_actual, direccion_actual = 0, 0, 0

for i in range(largo_pasillo):
    n = max(0, i - inicio_curva)
    offset = math.sin(n * 0.3) * amplitud * min(1.0, n/10)

    x_actual = offset
    y_actual += longitud_segmento
    posiciones.append((x_actual, y_actual, direccion_actual))

# ---------------- CREAR PAREDES (CUBOS) ----------------
for i in range(len(posiciones) - 1):
    x, y, angulo = posiciones[i]

    # Izquierda
    bpy.ops.mesh.primitive_cube_add(location=(x - ancho_pasillo, y, 1))
    pared_izq = bpy.context.active_object
    pared_izq.scale = (0.8, 0.8, 2)
    pared_izq.data.materials.append(mat_negro if i % 2 == 0 else mat_morado)

    # Derecha
    bpy.ops.mesh.primitive_cube_add(location=(x + ancho_pasillo, y, 1))
    pared_der = bpy.context.active_object
    pared_der.scale = (0.8, 0.8, 2)
    pared_der.data.materials.append(mat_morado if i % 2 == 0 else mat_negro)

# ---------------- CREAR SUELO CURVO ----------------
borde_izq = []
borde_der = []

for x, y, angulo in posiciones:
    borde_izq.append((x - ancho_pasillo + grosor_pared, y, 0))
    borde_der.append((x + ancho_pasillo - grosor_pared, y, 0))

mesh = bpy.data.meshes.new("SueloCurvo")
obj_suelo = bpy.data.objects.new("SueloCurvo", mesh)
bpy.context.collection.objects.link(obj_suelo)

bm = bmesh.new()
verts_izq = [bm.verts.new(p) for p in borde_izq]
verts_der = [bm.verts.new(p) for p in borde_der]

for i in range(len(verts_izq) - 1):
    bm.faces.new((verts_izq[i], verts_der[i],
                  verts_der[i + 1], verts_izq[i + 1]))

bm.to_mesh(mesh)
bm.free()

obj_suelo.data.materials.append(mat_suelo)

subsurf = obj_suelo.modifiers.new(name="Subdivision", type='SUBSURF')
subsurf.levels = 2
bpy.context.view_layer.objects.active = obj_suelo
bpy.ops.object.shade_smooth()

# ---------------- CÁMARA ANIMADA ----------------
bpy.ops.object.camera_add()
cam = bpy.context.active_object
cam.rotation_euler = (math.radians(90), 0, 0)

for f in range(1, 251):
    progreso = (f / 250) * (largo_pasillo - 1)

    n_cam = max(0, progreso - inicio_curva)
    offset_cam = math.sin(n_cam * 0.3) * amplitud * min(1.0, n_cam/10)

    cam.location.x = offset_cam
    cam.location.y = progreso * longitud_segmento
    cam.location.z = 1.5

    cam.keyframe_insert(data_path="location", frame=f)

# ---------------- LUZ ----------------
bpy.ops.object.light_add(type='SUN', location=(0, 0, 15))

```
<img width="835" height="550" alt="image" src="https://github.com/user-attachments/assets/826a0386-4b2c-4c4f-b7bf-e713f4472275" />


