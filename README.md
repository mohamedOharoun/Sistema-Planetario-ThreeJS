Sistema Solar Interactivo con Texturas e Iluminación
Autor: Mohamed O. Haroun Zarkik
Introducción
Este proyecto implementa una visualización tridimensional interactiva del Sistema Solar, demostrando conceptos fundamentales de renderizado gráfico como el uso de texturas, la iluminación realista y las sombras. Los cuerpos celestes introducidos son los planetas (por desgracia, Plutón se queda fuera) y sus respectivos satélites naturales. El código crea un entorno inmersivo donde el usuario puede explorar los planetas, sus lunas y experimentar diferentes perspectivas de observación. Este proyecto es una excelente introducción a técnicas avanzadas de visualización 3D.

Arquitectura del Sistema
Dependencias y Tecnologías
El proyecto está construido sobre Three.js (r128), una biblioteca JavaScript para renderizado 3D basada en WebGL. Los módulos principales utilizados son:

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
