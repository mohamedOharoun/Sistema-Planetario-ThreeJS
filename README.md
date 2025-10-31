# Sistema Solar Interactivo con Texturas e Iluminación

**Autor**: Mohamed O. Haroun Zarkik

## Introducción

Este proyecto implementa una visualización tridimensional interactiva del Sistema Solar, demostrando conceptos fundamentales de renderizado gráfico como el uso de texturas, la iluminación realista y las sombras. Los cuerpos celestes introducidos son los planetas (por desgracia, Plutón se queda fuera) y sus respectivos satélites naturales. El código crea un entorno inmersivo donde el usuario puede explorar los planetas, sus lunas y experimentar diferentes perspectivas de observación. Este proyecto es una excelente introducción a técnicas avanzadas de visualización 3D.

---

## Características Principales

### 1. Texturas y Materiales Realistas

Cada cuerpo celeste cuenta con texturas que le aportan realismo y detalle visual. El código carga imágenes de texturas desde archivos externos y las asigna a los materiales de cada objeto:

```javascript
const texture = textureLoader.load(data.texture);
const material = new THREE.MeshStandardMaterial({
  map: texture,
  roughness: 1.0,
  metalness: 0.0
});
```

Las texturas transforman esferas simples en representaciones visuales convincentes de planetas reales. Utilizar materiales estándar permite que las superficies interactúen realísticamente con la luz del sistema.

### 2. Iluminación y Sombras Realistas

El Sol actúa como fuente de luz puntual, emitiendo radiación que proyecta sombras sobre los planetas. Esta implementación crea profundidad visual y ayuda a entender las posiciones relativas de los objetos:

```javascript
const sunLight = new THREE.PointLight(0xffffff, 1.2, 0, 1.5);
sunLight.castShadow = true;
sunLight.shadow.mapSize.width = 2048;
sunLight.shadow.mapSize.height = 2048;
```

La luz ambiental suave complementa la iluminación principal, evitando que las caras oscuras queden completamente negras. Este balance es crucial para crear escenas tridimensionales convincentes.

### 3. Anillos de Saturno

Saturno se diferencia de otros planetas mediante la adición de anillos, generados como geometría independiente. Los anillos utilizan texturas especiales y se rotan con respecto al planeta:

```javascript
const ringGeometry = new THREE.RingGeometry(
  data.ringInner,
  data.ringOuter,
  64
);
const ring = new THREE.Mesh(ringGeometry, ringMaterial);
ring.rotation.x = Math.PI / 2;
```

---

## Elementos Interactivos

### Control de Velocidad Temporal

Un deslizador permite ajustar la velocidad de rotación y órbita de todos los cuerpos celestes en tiempo real, sin reiniciar la simulación:

```javascript
document.getElementById("speedSlider")
  .addEventListener("input", (e) => {
    timeScale = parseFloat(e.target.value);
  });
```

Esto permite acelerar o desacelerar la simulación para observar el movimiento orbital de manera clara.

### Cambio de Perspectiva de Cámara

El usuario puede alternar entre dos modos de visualización:

- **Vista General**: Perspectiva omnisciente con controles tipo órbita alrededor del sistema
- **Vista de Astronauta**: Cámara en primera persona, situándose en el espacio donde el usuario controla la rotación en el lugar y el movimiento hacia delante/atrás

```javascript
if (keys.left) rot.yaw += rotSpeed;
if (keys.up) astronaut.position.addScaledVector(forward, moveSpeed);
```

Esta funcionalidad proporciona dos formas complementarias de explorar el sistema.

### Selector de Cuerpos Celestes

Dos desplegables permiten seleccionar cualquier planeta o luna para centrar la cámara automáticamente sobre él:

```javascript
const planet = planets.find((p) => p.userData.name === val);
if (planet) focusOnObject(planet);
```

Al seleccionar un objeto, la cámara se posiciona estratégicamente para ofrecer una vista óptima del cuerpo elegido, manteniendo el resto del sistema visible pero atenuado.

### Alternar Órbitas

Un control de casilla permite mostrar u ocultar las líneas de órbita de todos los cuerpos:

```javascript
document.getElementById("showOrbits")
  .addEventListener("change", (e) => {
    orbits.forEach((orbit) => (orbit.visible = e.target.checked));
  });
```

Esta opción reduce el desorden visual cuando se desea una perspectiva más limpia del Sistema Solar.

---

## Elementos Visuales Especiales

### Nave Espacial Orbitante

Una nave espacial tridimensional recorre una órbita alrededor del sistema, demostrando cómo los objetos compuestos se pueden ensamblar a partir de geometrías simples. La nave incluye:

- **Fuselaje**: Cilindro rotado que forma el cuerpo
- **Cono de proa**: Define la dirección frontal de la nave
- **Cockpit**: Esfera semitransparente que simula una ventana
- **Alas y propulsores**: Detalles estructurales
- **Efectos de brillo**: Sprites aditivos que simulan el brillo de los motores

```javascript
const body = new THREE.Mesh(
  new THREE.CylinderGeometry(0.6, 0.6, 2.6, 16),
  bodyMat
);
body.rotation.z = Math.PI / 2;
```

### Ambiente Estelar

El fondo contiene dos capas de estrellas: una cercana generada con puntos, y otra distante con sprites suaves que simulan nebulosas. Esto proporciona contexto visual sin interferir con los cuerpos celestes principales.

---

## Estructura del Código

### Inicialización (`init`)

Configura la escena, carga texturas, crea la iluminación y posiciona todos los cuerpos celestes.

### Loop de Animación (`animate`)

Se ejecuta continuamente y actualiza:
- Posiciones orbitales según el tiempo simulado
- Rotaciones de planetas y lunas
- Movimiento de la cámara según entrada del usuario
- Renderizado final

### Gestión de Entrada

El código captura eventos de teclado (flechas direccionales) para permitir navegación fluida y controles intuitivos en ambos modos de cámara.

---

## Conceptos de Renderizado Aplicados

| Concepto | Descripción | Implementación |
|----------|-------------|-----------------|
| **Texturas** | Imágenes 2D aplicadas a superficies 3D | `TextureLoader` y propiedades `map` de materiales |
| **Iluminación Puntual** | Luz que emana de una posición específica | `PointLight` en la posición del Sol |
| **Sombras** | Oscurecimiento en superficies donde la luz está bloqueada | `castShadow` y `receiveShadow` en objetos |
| **Materiales Estándar** | Responden realísticamente a la luz ambiente y puntual | `MeshStandardMaterial` con propiedades de rugosidad |
| **Composición de Objetos** | Objetos complejos formados por geometrías simples | Grupos jerárquicos (astronauta, nave) |

---

## Conclusión

Este proyecto integra técnicas esenciales de gráficos tridimensionales en una aplicación interactiva y educativa. El resultado es un sistema accesible que demuestra cómo las texturas aportan realismo, cómo la iluminación y las sombras crean profundidad, y cómo la interacción enriquece la experiencia del usuario en entornos virtuales.

## Demo
A continuación una pequeña demostración de la aplicación en funcionamiento.

https://github.com/user-attachments/assets/0e17c657-6fe1-4006-a669-ce99c14ffbe6
