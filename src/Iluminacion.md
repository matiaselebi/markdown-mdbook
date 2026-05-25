# Iluminación

Una buena iluminación sirve para darle volumen a los objetos, evocar sensaciones, guiar la atención del jugador y darle información para tomar decisiones.

La luz puede ser directa, cuando viaja de la fuente al ojo sin tocar nada, o indirecta, cuando interactúa rebotando en una o más superficies antes de llegar a la cámara. Cuando un rayo de luz choca contra un material puede transmitirse, reflejarse, refractarse, difractarse, absorberse o dispersarse.

Existen dos formas principales de procesar la iluminación en un motor gráfico:
- Iluminación Baked: Se calcula antes de ejecutar el juego y se usa para objetos estáticos. El motor modifica las texturas de los materiales aplicandoles los colores de luz correspondientes.  
- Iluminación Realtime: Se calcula frame a frame durante la ejecución del juego. Se usa para luces y objetos dinámicos que se mueven. 

## Configuración de luces Baked

Para empezar, si estás en una escena de interior, tenés que oscurecer el entorno. Creá un nodo WorldEnvironment y bajale el Energy Multiplier a 0.

Para que los objetos reaccionen a este tipo de luz, todos los MeshInstance3D tienen que tener su propiedad Global Illumination en modo Static. Además, es obligatorio ir a las opciones de la malla y habilitar Add UV2 y desarrollar el UV2 para Lightmap/AO. Si importás un modelo externo, tenés que activar la opción Generate Lightmap UV2 desde la pestaña de importación.  

Si querés usar materiales que emitan luz propia, en su material tenés que activar Emission y asignarle un color y un Energy Multiplier. Para que el motor reconozca ese material como luz estática, poné la propiedad Global Illumination en modo StaticMaterial.  

Para procesar todo, agregá un nodo LightmapGI a la escena. En el inspector podés ajustar la calidad general en Quality, la cantidad de rebotes en Bounces y la resolución usando Texel Scale, donde un valor menor da más calidad pero consume más rendimiento. Finalmente, tocá el botón Bake Lightmaps y el motor generará las texturas de luz.  

## Light Probes

Los Light Probes guardan información de iluminación indirecta estática en distintos puntos del mapa. Sirven para que los objetos dinámicos, como el jugador, reciban luces y sombras realistas al moverse por el escenario sin necesidad de calcular todo en tiempo real. Se generan solos al hacer el bake si la opción Gen Probes dentro del LightmapGI tiene un valor distinto a Disabled.

## Luces en tiempo real y entornos
Godot ofrece tres nodos principales para luces dinámicas:
- OmniLight3D: Emite luz hacia todas las direcciones desde un punto central, como una bombilla.
- SpotLight3D: Proyecta luz en forma de cono, ideal para reflectores o linternas.
- DirectionalLight3D: Simula el sol y afecta a toda la escena según su dirección. Podés agregar una segunda luz de este tipo con menor energía y otro tono para simular rebotes.

Para que un objeto proyecte sombras dinámicas, la luz debe tener la opción Shadows activada y el objeto debe tener Cast Shadow en On.

Para simular un cielo, agregá un WorldEnvironment, poné el Background Mode en Sky y asignale un SkyMaterial. Podés usar un PanoramaSkyMaterial para una imagen HDRI realista, un ProceduralSkyMaterial para algo liviano y configurable, o un PhysicalSkyMaterial para simulación física.

## Creando una linterna funcional

Para hacer una linterna, creamos un nodo jugador y le agregamos un CSGCylinder3D rotado 90 grados en el eje X para que sea el modelo de la linterna. A este cilindro le agregamos un nodo SpotLight3D como hijo.

Al cilindro le asignamos este script para controlar la energía de la luz:
```python
 extends Node3D

@onready var spot_light_3d: SpotLight3D = $SpotLight3D
var encendida := false

func _ready():
    spot_light_3d.light_energy = 0.0

func interactuar_con_linterna():
    encendida = !encendida
    spot_light_3d.light_energy = 10.0 if !encendida else 0.0
``` 

Y en el script del jugador, detectamos el clic del mouse para llamar a esa función:

```python 
@onready var linterna: CSGCylinder3D = $Linterna

func _input(event):
    if linterna and event is InputEventMouseButton and event.pressed and event.button_index == MOUSE_BUTTON_LEFT:
        if linterna.has_method("interactuar_con_linterna"):
            linterna.interactuar_con_linterna()
``` 

## Parpadeo de luces (Flickering)
Podés simular luces rotas usando ondas matemáticas. A una OmniLight3D le agregamos este script que evalúa distintas formas de onda según el tiempo de ejecución:

```python 
extends Node3D

enum WaveForm { SIN, TRIANGLE, SQUARE, SAW, INVERSE, NOISE }
@export var luz: OmniLight3D
@export var frecuencia = 5.0
@export var amplitud = 1.0
@export var comienzo := 0.0
@export var fase = 0.0
@export var forma_onda: WaveForm = WaveForm.SIN

func _process(delta):
    if not luz:
        return
    var intensidad = evaluar_onda() * amplitud + comienzo
    luz.light_energy = clamp(intensidad, 0.0, 1.0)

func evaluar_onda() -> float:
    var x = (Time.get_ticks_msec() * 0.001 + fase) * frecuencia
    x -= floor(x)
    match forma_onda:
        WaveForm.SIN:
            return sin(x * PI * 2)
        WaveForm.TRIANGLE:
            return x * -1 if x < 0.5 else -4 * x + 3
        WaveForm.SQUARE:
            return 1.0 if x < 0.5 else -1.0
        WaveForm.SAW:
            return x
        WaveForm.INVERSE:
            return 1.0 - x
        WaveForm.NOISE:
            return randf() * 2 - 1
    return 1.0
``` 

## Post Procesamiento
Los efectos de postprocesado se aplican globalmente después de renderizar toda la escena usando un nodo WorldEnvironment con un recurso Environment asignado.

Algunos efectos comunes son el Glow para hacer brillar zonas intensas, Tonemap Exposure para la exposición general, Volumetric Fog para niebla realista y SSAO/SSR para sombras y reflejos avanzados. Otros efectos como el Vignette, el grano fílmico o la aberración cromática requieren usar shaders personalizados o complementos extra.

Podés activar estos efectos desde código accediendo al environment:

```python
extends WorldEnvironment

func _ready():
    environment.glow_enabled = true
    environment.glow_intensity = 2.0
    environment.glow_strength = 2.0 
``` 

Como los efectos son globales, si querés que solo ocurran en una zona específica, tenés que usar un Area3D que detecte al jugador y modifique las opciones por código al entrar y salir.

Le pasamos el WorldEnvironment como variable exportada al Area3D y usamos este script:
```python 
extends Area3D

@export var world_enviroment: WorldEnvironment
var env: Environment

func _ready() -> void:
    env = world_enviroment.environment
    env.glow_enabled = false
    body_entered.connect(_on_area_body_entered)
    body_exited.connect(_on_area_body_exited)

func _on_area_body_entered(body):
    if body.name == "Jugador":
        env.glow_enabled = true

func _on_area_body_exited(body):
    if body.name == "Jugador":
        env.glow_enabled = false
``` 