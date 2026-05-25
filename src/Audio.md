# Audio en Godot
El audio es un elemento esencial para cualquier videojuego. Para que un sonido se reproduzca se necesitan 3 objetos principales.

El Stream es el archivo de sonido a reproducir, como un ogg, wav o mp3. Se lo asignas al reproductor.
El AudioStreamPlayer es el nodo que reproduce el sonido. Puede ser global y sin posicion, 2D o 3D. Aca definis el volumen y el pitch.
El AudioListener se encarga de escuchar la fuente de sonido para que salga por los parlantes y generalmente se agrega a la camara.

## Configuracion en el editor
Tenes que crear una carpeta Sonidos y arrastrar dos archivos mp3. En este caso un efecto llamado Monedita para cuando el personaje salte y una cancion llamada musica para el fondo.

Creamos un proyecto con un cubo transformado en piso y una esfera como jugador. Despues armamos una escena nueva con un nodo vacio llamado AudioController. A ese nodo le agregamos dos AudioStreamPlayer, uno llamado MusicPlayer y otro SfxPlayer.

Le agregamos este script al AudioController:
println!("pongo python por que si pongo que el codigo es GDScript no se pinta");
```python 
extends Node
@onready var music_player: AudioStreamPlayer = $MusicPlayer
@onready var sfx_player: AudioStreamPlayer = $SfxPlayer

func play_music():
    if not music_player.playing:
        music_player.play()

func play_sfx():
    sfx_player.play()
``` 
## Singleton con AutoLoad
AutoLoad es un sistema que permite cargar un script o escena al inicio y mantenerlo vivo durante toda la ejecucion. Esto evita duplicar logica y te deja acceder al controlador desde cualquier lado.

Para configurarlo vas a Project, despues Project Settings y buscas la pestaña Globales o AutoLoad. Ahi agregas la escena AudioController y le asignas un nombre global.

Una vez hecho eso, podes llamar a los sonidos desde el script del jugador de esta forma:
```python 
if can_jump:
    if Input.is_action_just_pressed(input_jump) and is_on_floor():
        velocity.y = jump_velocity
        AudioController.play_sfx()

func _ready():
    if not music_player.playing:
        AudioController.play_music()
```

Para sonidos espaciales existen los nodos AudioListener3D y AudioStreamPlayer3D, donde el volumen y paneo cambian segun donde este el objeto en el mundo. Tambien podes usar la funcion Playlist para reproducir varios sonidos en orden o aleatorio, y la funcion Interactive para controlar pistas por separado.