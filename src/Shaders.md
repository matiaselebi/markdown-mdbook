# Shaders

Un shader es un programa que se ejecuta en la tarjeta gráfica o GPU. Su objetivo original era la representación de gráficos, pero hoy en día se usan para optimizar el rendimiento, diferenciar visualmente un juego y ayudar al equipo de arte a crear efectos complejos sin tener que animar todo a mano.

Se ejecutan en la GPU porque este hardware permite una alta concurrencia. Por ejemplo, para renderizar un juego a 60 cuadros por segundo en resolucion 4K, el motor tiene menos de 16,6 milisegundos para procesar mas de 8 millones de pixeles por frame. La GPU logra esto teniendo un acceso veloz a los datos y usando miles de nucleos que procesan shaders de forma concurrente.

El encargado de trabajar con esto suele ser el Technical Artist, un perfil hibrido entre artista y programador. Su trabajo consiste en desarrollar tecnologia de shaders, iluminacion y postprocesado, asegurando que el juego se vea increible sin arruinar el rendimiento. Un ejemplo claro de esto son los materiales complejos de The Last of Us Part II, donde usan shaders para mezclar texturas de pintura descascarada, cemento y humedad en tiempo real.

## Tipos de shaders y lenguajes

Existen varios tipos de shaders, pero los minimos necesarios para renderizar son dos:

Vertex Shaders: corren como minimo una vez por cada vertice del modelo. Se ejecutan antes de la rasterizacion.

Fragment Shaders o Pixel Shaders: corren como minimo una vez por cada pixel en pantalla. Se ejecutan despues de la rasterizacion y definen el color final de cada pixel.

Existen otros como los Compute Shaders para calculos matematicos fuera del pipeline grafico, o los Tessellation Shaders para subdividir geometria, aunque estos ultimos no son soportados directamente por Godot de la misma forma.

Cada API grafica tiene su lenguaje. OpenGL usa GLSL, Vulkan usa SPIR-V y DirectX usa HLSL. Godot utiliza su propio lenguaje llamado Godot Shader Language o GSL, que esta adaptado a su arquitectura pero es muy similar a GLSL.

## Materiales y tipos de shaders

Un material es un asset que combina un shader, una textura y parametros de control. En Godot existen distintos tipos principales.

StandardMaterial3D: es el material por defecto para 3D y utiliza PBR o Physically Based Rendering. Esto significa que busca el fotorealismo haciendo que las superficies conserven la energia y no reflejen mas luz de la que reciben.

ORMMaterial3D: es una variante optimizada del Standard que agrupa la oclusion, la rugosidad y el aspecto metalico en una sola imagen para ahorrar recursos.

CanvasItemMaterial: se usa para objetos 2D y permite aplicar efectos como modos de mezcla y filtros.

ShaderMaterial: permite usar shaders personalizados escritos en codigo.

Si alguna vez ves que tu modelo 3D se pone de color magenta brillante, significa que el shader tiene un error de sintaxis en el código y el motor no lo pudo compilar.

Para empezar a escribir un shader personalizado, la estructura básica define primero el tipo en la primera linea. Puede ser spatial para 3D, canvas_item para 2D, sky para el cielo o particles.

```python 
shader_type spatial;
render_mode unshaded;

uniform sampler2D my_texture : hint_albedo;

void fragment() {
    vec4 tex_color = texture(my_texture, UV);
    ALBEDO = tex_color.rgb;
}
``` 

## Visual Shaders

Si no querés escribir código puro, Godot incluye los Visual Shaders. Es un editor basado en nodos conectables pensado para artistas. Para usarlo tenés que crear un recurso de tipo VisualShader en lugar de un ShaderMaterial normal.

Por ejemplo, para hacer un efecto de contorno u outline en un objeto, le asignás a tu mesh un StandardMaterial3D. En la sección Next Pass le agregás un nuevo shader visual. Adentro de la vista de Vertex multiplicás el nodo Normal por un parámetro float en 0.05 y se lo sumás al vértice original. En la vista de Fragment le pasás un parámetro de color directamente al Albedo.

Finalmente, en las opciones generales de ese shader visual, cambiás el modo Cull a Front para que las caras se rendericen al revés y generen el borde de color.

## Shader de agua por codigo

Para hacer un efecto de agua con movimiento matemático constante creamos un shader común y usamos este código.

```python 
shader_type spatial;

uniform sampler2D water_normal_noise;
uniform vec3 water_color;

void fragment() {
    vec2 _uv = UV;
    
    _uv.x += sin(TIME + (_uv.x + _uv.y) * 0.25) * 0.01;
    _uv.y += cos(TIME + (_uv.x - _uv.y) * 0.25) * 0.01;
    
    ALBEDO = water_color;
    NORMAL_MAP = texture(water_normal_noise, _uv).rgb;
    NORMAL *= 0.5;
    ROUGHNESS = 0.2;
}
``` 
Una vez guardado el script, vas al inspector y en la pestaña Shader Parameters le asignás el color de agua que más te guste y le cargás una textura de ruido. A esa textura le ponés el tipo FastNoise, activás la opción Ridged en la sección Fractal y te asegurás de marcar las casillas Seamless y As Normal Map para que el relieve funcione correctamente y se repita sin cortes.