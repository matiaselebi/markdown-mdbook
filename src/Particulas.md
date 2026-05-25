# Sistema de partículas
Un sistema de particulas es un nodo que genera y gestiona automaticamente muchos objetos pequeños. Cada uno tiene sus propias propiedades de posicion, velocidad, color y tiempo de vida.

En Godot existen dos tipos principales. GPUParticles corre en la tarjeta grafica y permite miles de particulas, pero no funciona en juegos web. CPUParticles corre en el procesador, soporta menos particulas pero es compatible con web y moviles.

## Creando una explosion
Vamos a crear una explosion dividida en tres partes, empezando por los fragmentos. En el nodo principal agregamos un GPUParticles3D y lo llamamos Fragmentos. Ponemos la propiedad amount a 30, el time a 0.3 y explosiveness a 1.

En la seccion Draw Passes agregamos un PrismMesh. Entramos a su material, creamos un StandardMaterial y le ponemos color naranja en el albedo. Para que brille, activamos emission, subimos el energy multiplier a 1.75 y le ponemos el mismo color naranja.

En Process Material agregamos un ParticleProcessMaterial. Adentro de Spawn velocity ponemos el Spread a 180, initial velocity min en 15 y max en 18. En Particle Flags marcamos Align Y. Por ultimo en Scale ponemos el minimo en 0.05, el maximo en 0.85 y le agregamos una curva de desvanecimiento.

## Fuego
Agregamos otro GPUParticles3D llamado Fuego. En Draw Passes le ponemos un QuadMesh y creamos un material con estas caracteristicas.

Transparency en Alpha.
Blend mode en Add.
Shading mode en unshaded.
Albedo color anaranjado rojizo.
Vertex color en Use as Albedo.
Billboard mode en particle billboard.

Cambiamos amount a 180, lifetime a 0.5, y ponemos randomness y explosiveness en 1. En drawing activamos Local Coords y ponemos Draw Order en View Depth. Le agregamos un ParticleProcessMaterial, dejamos la gravedad en 0, ponemos velocity min en 5, max en 10 y spread a 180.

## Humo
Copiamos el nodo de fuego y le ponemos Humo. Aca es muy importante hacer click derecho en Process Material y elegir Hacer unico para no modificar el fuego sin querer.

Cambiamos el amount a 25 y el Draw Pass a un SphereMesh. En el material cambiamos el blend mode a Add o Substract y el albedo a negro. En el process material nuevo cambiamos la gravedad a Y en 10 y Z en 10 para que el humo suba.

Activacion por codigo
Desactivamos la opcion emitting y activamos one shot en los tres nodos de particulas. Despues le agregamos este script al nodo principal para que detone al apretar la barra espaciadora.
```python 
extends Node3D
@onready var fragmentos: GPUParticles3D = $Fragmentos
@onready var fuego: GPUParticles3D = $Fuego
@onready var humo: GPUParticles3D = $Humo

func _process(delta):
    if Input.is_key_pressed(KEY_SPACE):
        _explosion()

func _explosion():
    fragmentos.emitting = true
    fuego.emitting = true
    humo.emitting = true
```