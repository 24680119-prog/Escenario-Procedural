## Escenario Procedural en Blender 

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

**mat_negro, mat_morado, mat_suelo:**Materiales creados a partir de la función definida.

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
offset: Desplazamiento horizontal en el eje X.
y_actual += longitud_segmento: Hace avanzar el pasillo.
append(): Guarda cada nueva posición en la lista.



