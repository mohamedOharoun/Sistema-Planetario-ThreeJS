# Sistema Solar Interactivo con Texturas e Iluminación

**Autor**: Mohamed O. Haroun Zarkik

En el siguiente <a href="https://codesandbox.io/p/sandbox/956yl2" target="_blank">enlace</a> se puede ver el proyecto en CodeSandBox

## Introducción

Este proyecto implementa una visualización tridimensional interactiva del Sistema Solar, demostrando conceptos fundamentales de renderizado gráfico como el uso de texturas, la iluminación realista y las sombras. Los cuerpos celestes introducidos son los planetas (por desgracia, Plutón se queda fuera) y sus respectivos satélites naturales. El código crea un entorno inmersivo donde el usuario puede explorar los planetas, sus lunas y experimentar diferentes perspectivas de observación.

---

## Arquitectura del Sistema

### Dependencias y Tecnologías

El proyecto está construido sobre **Three.js** (r128), una biblioteca JavaScript para renderizado 3D basada en WebGL. Los módulos principales utilizados son:

```javascript
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
```

**Three.js** abstrae la complejidad de WebGL, proporcionando una API intuitiva para crear escenas tridimensionales. `OrbitControls` permite navegación orbital intuitiva mediante mouse o trackpad.

### Estructura de Datos Astronómicos

El sistema utiliza un objeto de configuración central que define las propiedades de cada cuerpo celeste:

```javascript
const solarSystemData = {
  mercury: {
    radius: 0.4,           // Radio visual
    distance: 6,           // Distancia orbital desde el Sol
    orbitalPeriod: 0.24,   // Período orbital en años terrestres
    rotationPeriod: 58.6,  // Período de rotación en días terrestres
    texture: "./textures/mercury_texture.jpg",
    moons: []
  },
  // ... más planetas
};
```

Las escalas están ajustadas para visualización (no son proporciones reales) manteniendo las relaciones relativas entre planetas. Los valores negativos en `rotationPeriod` indican rotación retrógrada (Venus, Urano).

### Inicialización de la Escena

La función `init()` configura todos los elementos fundamentales:

```javascript
function init() {
  // 1. Escena y fondo
  scene = new THREE.Scene();
  scene.background = new THREE.Color(0x000000);
  
  // 2. Cámara perspectiva
  camera = new THREE.PerspectiveCamera(
    60,                                    // FOV
    window.innerWidth / window.innerHeight, // Aspect ratio
    0.1,                                   // Near plane
    2000                                   // Far plane
  );
  camera.position.set(0, 50, 80);
  
  // 3. Renderer con sombras
  renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.PCFSoftShadowMap;
  
  // 4. Controles orbitales
  controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;
}
```

El **damping** suaviza el movimiento de los controles, creando una experiencia de navegación más natural y fluida.

---

## Características Principales

### 1. Texturas y Materiales Realistas

Cada cuerpo celeste cuenta con texturas que le aportan realismo y detalle visual. El código carga imágenes de texturas desde archivos externos y las asigna a los materiales de cada objeto:

```javascript
const textureLoader = new THREE.TextureLoader();
const texture = textureLoader.load(data.texture);
const material = new THREE.MeshStandardMaterial({
  map: texture,
  roughness: 1.0,  // Superficie mate (no reflectante)
  metalness: 0.0   // Sin propiedades metálicas
});
```

**MeshStandardMaterial** es un material basado en principios físicos (PBR - Physically Based Rendering) que responde de manera realista a la iluminación. Los parámetros `roughness` y `metalness` controlan cómo la luz interactúa con la superficie:

- **Roughness = 1.0**: Superficie completamente difusa (como roca o tierra)
- **Metalness = 0.0**: Material dieléctrico (no conductor)

### 2. Sistema de Iluminación Realista

#### Luz Puntual del Sol

El Sol actúa como fuente de luz puntual, emitiendo radiación que proyecta sombras sobre los planetas:

```javascript
const sunLight = new THREE.PointLight(0xffffff, 1.2, 0, 1.5);
sunLight.castShadow = true;
sunLight.shadow.mapSize.width = 2048;
sunLight.shadow.mapSize.height = 2048;
sunLight.shadow.camera.near = 0.5;
sunLight.shadow.camera.far = 500;
sun.add(sunLight);
```

Parámetros clave:
- **Intensidad**: 1.2 (ligeramente más brillante que la luz estándar)
- **Decay**: 1.5 (atenuación realista con la distancia siguiendo ley del cuadrado inverso)
- **Shadow map**: 2048x2048 para sombras de alta calidad

#### Luz Ambiental

```javascript
const ambientLight = new THREE.AmbientLight(0x222222);
scene.add(ambientLight);
```

La luz ambiental suave (color oscuro) evita que las caras no iluminadas queden completamente negras, simulando la dispersión de luz en el espacio.

### 3. Sistema de Órbitas

Cada planeta sigue una trayectoria circular alrededor del Sol. Las órbitas se calculan usando trigonometría básica:

```javascript
const angle = (timestamp / data.orbitalPeriod) * Math.PI * 2;
planet.position.x = Math.cos(angle) * data.distance;
planet.position.z = Math.sin(angle) * data.distance;
```

Las líneas de órbita se crean con geometría de línea:

```javascript
const orbitGeometry = new THREE.BufferGeometry();
const orbitPoints = [];
for (let i = 0; i <= 64; i++) {
  const angle = (i / 64) * Math.PI * 2;
  orbitPoints.push(
    Math.cos(angle) * data.distance,
    0,
    Math.sin(angle) * data.distance
  );
}
orbitGeometry.setAttribute(
  'position',
  new THREE.Float32BufferAttribute(orbitPoints, 3)
);
const orbit = new THREE.Line(orbitGeometry, orbitMaterial);
```

### 4. Anillos de Saturno

Saturno se diferencia mediante anillos generados como geometría independiente:

```javascript
const ringGeometry = new THREE.RingGeometry(
  data.ringInner,  // Radio interior
  data.ringOuter,  // Radio exterior
  64               // Segmentos
);
const ringTexture = textureLoader.load(data.ringTexture);
const ringMaterial = new THREE.MeshBasicMaterial({
  map: ringTexture,
  side: THREE.DoubleSide,  // Visible desde ambos lados
  transparent: true,
  opacity: 0.8
});
const ring = new THREE.Mesh(ringGeometry, ringMaterial);
ring.rotation.x = Math.PI / 2;  // Rotar a plano horizontal
planet.add(ring);
```

Al agregar el anillo como hijo del planeta, hereda automáticamente su transformación (posición, rotación).

### 5. Sistema de Satélites Naturales

Las lunas orbitan sus planetas padre utilizando el mismo principio de coordenadas polares:

```javascript
function createMoon(planet, moonData) {
  const moon = new THREE.Mesh(moonGeometry, moonMaterial);
  moon.userData = {
    name: moonData.name,
    distance: moonData.distance,
    period: moonData.period,
    parent: planet
  };
  planet.add(moon);  // Jerarquía: luna es hija del planeta
  
  // En el loop de animación:
  const angle = (timestamp / Math.abs(data.period)) * Math.PI * 2;
  moon.position.x = Math.cos(angle) * data.distance;
  moon.position.z = Math.sin(angle) * data.distance;
}
```

Al añadir la luna al planeta (`planet.add(moon)`), su sistema de coordenadas es relativo al planeta, simplificando enormemente la lógica orbital.

---

## Elementos Interactivos

### Control de Velocidad Temporal

Un deslizador HTML permite ajustar la velocidad de simulación en tiempo real:

```javascript
document.getElementById("speedSlider").addEventListener("input", (e) => {
  timeScale = parseFloat(e.target.value);
  document.getElementById("speedValue").textContent = timeScale.toFixed(1) + "x";
});

// Aplicación en el loop de animación:
timestamp = (Date.now() - t0) * 0.00001 * timeScale;
planet.rotation.y += (0.01 / data.rotationPeriod) * timeScale;
```

El `timeScale` multiplica tanto el tiempo de simulación como las velocidades de rotación, acelerando o ralentizando todo el sistema de forma coherente.

### Alternancia de Líneas Orbitales

```javascript
document.getElementById("showOrbits").addEventListener("change", (e) => {
  orbits.forEach(orbit => orbit.visible = e.target.checked);
});
```

Alternar la visibilidad de las órbitas reduce el desorden visual, permitiendo apreciar mejor los cuerpos celestes.

### Sistema Dual de Cámara

El proyecto implementa dos modos de visualización completamente distintos:

#### Vista General (Orbital)

Modo por defecto con controles tipo órbita:

```javascript
if (cameraMode === "general") {
  const forward = new THREE.Vector3();
  camera.getWorldDirection(forward);
  forward.y = 0;
  forward.normalize();
  
  const right = new THREE.Vector3();
  right.crossVectors(forward, new THREE.Vector3(0, 1, 0)).normalize();
  
  if (keys.up) camera.position.addScaledVector(forward, moveSpeed * 4);
  if (keys.left) camera.position.addScaledVector(right, -moveSpeed * 4);
  
  // CRÍTICO: Mover el target con la cámara
  const ahead = camera.position.clone().addScaledVector(forward, 50);
  controls.target.lerp(ahead, 0.25);
  controls.update();
}
```

La clave aquí es mantener el `controls.target` delante de la cámara para evitar que OrbitControls trate de orbitar alrededor de un punto remoto.

#### Vista de Astronauta (Primera Persona)

Cámara en primera persona con rotación en el lugar:

```javascript
if (cameraMode === "astronaut") {
  controls.enabled = false;  // Deshabilitar OrbitControls
  
  if (!astronaut.rotationData) astronaut.rotationData = { yaw: 0 };
  const rot = astronaut.rotationData;
  
  // Rotación izquierda/derecha cambia yaw
  if (keys.left) rot.yaw += rotSpeed;
  if (keys.right) rot.yaw -= rotSpeed;
  
  // Dirección de avance basada en yaw
  const forward = new THREE.Vector3(
    -Math.sin(rot.yaw),
    0,
    -Math.cos(rot.yaw)
  ).normalize();
  
  // Movimiento adelante/atrás
  if (keys.up) astronaut.position.addScaledVector(forward, moveSpeed);
  if (keys.down) astronaut.position.addScaledVector(forward, -moveSpeed);
  
  // Posicionar cámara en la cabeza
  camera.position.copy(astronaut.position).add(new THREE.Vector3(0, 0.7, 0));
  camera.lookAt(camera.position.clone().add(forward));
}
```

### Selectores de Objetos Celestes

Los menús desplegables permiten centrar la cámara en cualquier planeta o luna:

```javascript
function populateSelectors() {
  const planetSel = document.getElementById("planetSelector");
  const planetOptions = ["Sun", ...planets.map(p => p.userData.name)];
  
  planetOptions.forEach(name => {
    const opt = document.createElement("option");
    opt.value = name;
    opt.textContent = name.charAt(0).toUpperCase() + name.slice(1);
    planetSel.appendChild(opt);
  });
  
  planetSel.addEventListener("change", (e) => {
    const planet = planets.find(p => p.userData.name === e.target.value);
    if (planet) focusOnObject(planet);
  });
}
```

### Sistema de Enfoque Cinemático

Al seleccionar un objeto, la cámara se posiciona estratégicamente:

```javascript
function focusOnObject(obj) {
  focusedObject = obj;
  
  // Atenuar otros objetos sin ocultarlos
  planets.concat(moons).forEach(o => {
    if (o !== obj) {
      o.material.transparent = true;
      o.material.opacity = 0.2;
    } else {
      o.material.opacity = 1.0;
    }
  });
  
  // Calcular distancia apropiada
  const radius = obj.geometry.parameters.radius;
  focusDistance = moons.includes(obj) ? radius * 30 : radius * 12;
  
  // Posicionar desde el lado iluminado
  const target = new THREE.Vector3();
  obj.getWorldPosition(target);
  
  const sunPos = new THREE.Vector3();
  sun.getWorldPosition(sunPos);
  
  const sunToObj = new THREE.Vector3().subVectors(target, sunPos).normalize();
  const cameraDir = sunToObj.clone().negate();
  const cameraPos = target.clone().addScaledVector(cameraDir, focusDistance);
  
  camera.position.copy(cameraPos);
  controls.target.copy(target);
}
```

El enfoque cinemático continúa en el loop de animación:

```javascript
if (focusedObject) {
  // Cancelar si el usuario toma control manual
  if (keys.up || keys.down || keys.left || keys.right) {
    focusedObject = null;
  } else {
    const target = new THREE.Vector3();
    focusedObject.getWorldPosition(target);
    
    const currentDir = new THREE.Vector3()
      .subVectors(camera.position, controls.target)
      .normalize();
    
    const desiredPos = target.clone().addScaledVector(currentDir, focusDistance);
    
    // Seguimiento suave
    camera.position.lerp(desiredPos, 0.03);
    controls.target.lerp(target, 0.1);
  }
}
```

---

## Elementos Visuales Especiales

### Modelo del Astronauta

El astronauta se construye jerárquicamente a partir de geometrías primitivas:

```javascript
function createAstronaut() {
  const astronautGroup = new THREE.Group();
  
  // Cuerpo - cilindro vertical
  const body = new THREE.Mesh(
    new THREE.CylinderGeometry(0.15, 0.15, 0.8, 16),
    new THREE.MeshPhongMaterial({ color: 0xeeeeee })
  );
  astronautGroup.add(body);
  
  // Casco - esfera semitransparente
  const head = new THREE.Mesh(
    new THREE.SphereGeometry(0.18, 16, 16),
    new THREE.MeshPhongMaterial({
      color: 0x88ccff,
      transparent: true,
      opacity: 0.7,
      shininess: 100
    })
  );
  head.position.y = 0.35;
  astronautGroup.add(head);
  
  // Mochila, brazos, piernas...
  // (código completo en el documento)
  
  astronautGroup.position.set(0, 0, 70);
  astronautGroup.scale.set(2, 2, 2);
  scene.add(astronautGroup);
}
```

La transparencia del casco simula un visor de cristal, mientras que el `shininess` elevado crea reflejos especulares.

### Nave Espacial Orbitante

Una nave compleja ensamblada a partir de primitivas:

```javascript
function createSpaceMachine() {
  const ship = new THREE.Group();
  
  // Fuselaje - cilindro horizontal
  const body = new THREE.Mesh(
    new THREE.CylinderGeometry(0.6, 0.6, 2.6, 16),
    new THREE.MeshPhongMaterial({ color: 0xaaaaaa, shininess: 30 })
  );
  body.rotation.z = Math.PI / 2;
  ship.add(body);
  
  // Cono de proa
  const nose = new THREE.Mesh(
    new THREE.ConeGeometry(0.6, 1.0, 16),
    bodyMaterial
  );
  nose.rotation.z = Math.PI / 2;
  nose.position.x = 1.8;
  ship.add(nose);
  
  // Cabina semitransparente
  const cockpit = new THREE.Mesh(
    new THREE.SphereGeometry(0.45, 12, 12),
    new THREE.MeshPhongMaterial({
      color: 0x334455,
      transparent: true,
      opacity: 0.95,
      shininess: 80
    })
  );
  cockpit.position.x = 0.7;
  cockpit.position.y = 0.25;
  ship.add(cockpit);
  
  // Alas, propulsores, efectos de brillo...
  // (ver código completo)
  
  return ship;
}
```

#### Efectos de Propulsión

Los motores tienen sprites aditivos que simulan brillo:

```javascript
// Textura radial procedural
function createRadialTexture(size = 128, innerColor, outerColor) {
  const canvas = document.createElement('canvas');
  canvas.width = canvas.height = size;
  const ctx = canvas.getContext('2d');
  const grad = ctx.createRadialGradient(size/2, size/2, 0, size/2, size/2, size/2);
  grad.addColorStop(0, innerColor);
  grad.addColorStop(0.6, outerColor);
  ctx.fillStyle = grad;
  ctx.fillRect(0, 0, size, size);
  return new THREE.CanvasTexture(canvas);
}

// Sprite de motor con blending aditivo
const glowTex = createRadialTexture(256, "rgba(255,200,120,1)", "rgba(255,200,120,0)");
const glowMat = new THREE.SpriteMaterial({
  map: glowTex,
  color: 0xffb86b,
  transparent: true,
  blending: THREE.AdditiveBlending,
  depthWrite: false
});
const engineGlow = new THREE.Sprite(glowMat);
engineGlow.scale.set(1.2, 1.2, 1.2);
ship.add(engineGlow);
```

El **blending aditivo** suma los colores al framebuffer, creando efectos de luz brillante.

#### Movimiento Orbital

```javascript
function updateSpaceMachine(delta) {
  shipAngle += delta * 0.2 * timeScale;
  const radius = 45;
  const height = Math.sin(shipAngle * 0.5) * 10;
  
  spaceMachine.position.set(
    Math.cos(shipAngle) * radius,
    height,
    Math.sin(shipAngle) * radius
  );
  
  // Orientar hacia el centro
  spaceMachine.lookAt(0, 0, 0);
  spaceMachine.rotateY(Math.PI);  // Girar 180° para apuntar hacia adelante
}
```

### Campo Estelar de Fondo

Dos capas de estrellas crean profundidad:

#### Capa Cercana (Puntos)

```javascript
const starsGeometry = new THREE.BufferGeometry();
const starsVertices = [];
for (let i = 0; i < 5000; i++) {
  const x = (Math.random() - 0.5) * 2000;
  const y = (Math.random() - 0.5) * 2000;
  const z = (Math.random() - 0.5) * 2000;
  starsVertices.push(x, y, z);
}
starsGeometry.setAttribute(
  'position',
  new THREE.Float32BufferAttribute(starsVertices, 3)
);
const stars = new THREE.Points(starsGeometry, starsMaterial);
```

#### Capa Lejana (Nebulosas)

```javascript
function createFarStars() {
  // Estrellas distantes en esfera de radio 800-2200
  const farCount = 10000;
  const positions = new Float32Array(farCount * 3);
  const radiusMin = 800, radiusMax = 2200;
  
  for (let i = 0; i < farCount; i++) {
    const r = radiusMin + Math.random() * (radiusMax - radiusMin);
    const theta = Math.random() * Math.PI * 2;
    const phi = Math.acos(2 * Math.random() - 1);
    
    positions[i*3] = r * Math.sin(phi) * Math.cos(theta);
    positions[i*3+1] = r * Math.sin(phi) * Math.sin(theta);
    positions[i*3+2] = r * Math.cos(phi);
  }
  
  const farPoints = new THREE.Points(farGeom, farStarsMat);
  farPoints.frustumCulled = false;  // Siempre visible
  scene.add(farPoints);
  
  // Sprites de nebulosas gigantes
  for (let i = 0; i < 12; i++) {
    const sprite = new THREE.Sprite(nebulaeMaterial);
    sprite.position.set(/* posición aleatoria */);
    sprite.scale.set(300 + Math.random() * 800, ...);
    scene.add(sprite);
  }
}
```

---

## Loop de Animación

El ciclo principal actualiza toda la escena cada frame (~60 FPS):

```javascript
function animate() {
  requestAnimationFrame(animate);
  
  timestamp = (Date.now() - t0) * 0.00001 * timeScale;
  
  // 1. Rotación solar
  sun.rotation.y += 0.001;
  
  // 2. Actualizar planetas
  planets.forEach(planet => {
    const data = planet.userData;
    
    // Posición orbital
    const angle = (timestamp / data.orbitalPeriod) * Math.PI * 2;
    planet.position.x = Math.cos(angle) * data.distance;
    planet.position.z = Math.sin(angle) * data.distance;
    
    // Rotación propia
    planet.rotation.y += (0.01 / data.rotationPeriod) * timeScale;
  });
  
  // 3. Actualizar lunas
  moons.forEach(moon => {
    const data = moon.userData;
    const angle = (timestamp / Math.abs(data.period)) * Math.PI * 2;
    moon.position.x = Math.cos(angle) * data.distance;
    moon.position.z = Math.sin(angle) * data.distance;
    moon.rotation.y += 0.01 * timeScale;
  });
  
  // 4. Movimiento de cámara
  updateCameraMovement();
  
  // 5. Actualizar nave espacial
  updateSpaceMachine(0.016);
  
  // 6. Sistema de enfoque
  if (focusedObject) { /* ... */ }
  
  // 7. Visibilidad del astronauta
  if (astronaut) astronaut.visible = !isAstronautView;
  
  // 8. Actualizar controles y renderizar
  controls.update();
  renderer.render(scene, camera);
}
```

---

## Gestión de Entrada

El sistema de input captura eventos de teclado para navegación:

```javascript
let keys = { up: false, down: false, left: false, right: false };

document.addEventListener('keydown', (e) => {
  if (['ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight'].includes(e.key)) {
    e.preventDefault();  // Evitar scroll de página
  }
  
  const key = e.key.toLowerCase();
  if (key === 'arrowup') keys.up = true;
  if (key === 'arrowdown') keys.down = true;
  if (key === 'arrowleft') keys.left = true;
  if (key === 'arrowright') keys.right = true;
});

document.addEventListener('keyup', (e) => {
  const key = e.key.toLowerCase();
  if (key === 'arrowup') keys.up = false;
  if (key === 'arrowdown') keys.down = false;
  if (key === 'arrowleft') keys.left = false;
  if (key === 'arrowright') keys.right = false;
});
```

Este patrón permite movimiento suave y respuesta inmediata a múltiples teclas presionadas simultáneamente.

---

## Desafíos Técnicos y Soluciones

### 1. Conflicto entre Modos de Cámara y Sistema de Enfoque

**Problema**: El sistema de enfoque cinemático (`focusOnObject`) entraba en conflicto con el modo de astronauta, causando comportamientos impredecibles donde la cámara intentaba seguir un objeto mientras el usuario trataba de controlar manualmente el movimiento.

**Solución Implementada**:

```javascript
if (focusedObject) {
  // Detectar entrada del usuario y cancelar enfoque automáticamente
  if (keys.up || keys.down || keys.left || keys.right) {
    focusedObject = null;  // Usuario toma control
  } else {
    // Continuar seguimiento cinemático suave
    camera.position.lerp(desiredPos, 0.03);
    controls.target.lerp(target, 0.1);
  }
}
```

Esta solución prioriza el control del usuario: cualquier pulsación de tecla cancela inmediatamente el enfoque automático, devolviendo el control total al jugador.

### 2. Control de Movimiento en Vista de Astronauta

**Problema**: Implementar rotación en el lugar (yaw) mientras se mantiene movimiento adelante/atrás relativo a la dirección de visión era complejo. Los controles OrbitControls interfieren con este tipo de navegación en primera persona.

**Solución**: Deshabilitar completamente OrbitControls en modo astronauta y gestionar manualmente la orientación. Sin embargo, el resultado sigue siendo deficiente:

```javascript
if (cameraMode === "astronaut") {
  controls.enabled = false;
  
  // Almacenar estado de rotación en el objeto astronauta
  if (!astronaut.rotationData) astronaut.rotationData = { yaw: 0 };
  const rot = astronaut.rotationData;
  
  // Rotación pura (sin traslación)
  if (keys.left) rot.yaw += rotSpeed;
  if (keys.right) rot.yaw -= rotSpeed;
  
  // Vector de dirección basado en yaw
  const forward = new THREE.Vector3(
    -Math.sin(rot.yaw),
    0,
    -Math.cos(rot.yaw)
  ).normalize();
  
  // Movimiento en dirección de visión
  if (keys.up) astronaut.position.addScaledVector(forward, moveSpeed);
  if (keys.down) astronaut.position.addScaledVector(forward, -moveSpeed);
  
  // Mantener astronauta al nivel del suelo
  astronaut.position.y = 0;
  
  // Sincronizar cámara con astronauta
  camera.position.copy(astronaut.position).add(new THREE.Vector3(0, 0.7, 0));
  camera.lookAt(camera.position.clone().add(forward));
}
```
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
