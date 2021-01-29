---
description: >-
  Dans ce chapitre, nous allons voir les différents points pour correctement
  configurer son environnement pour développer des applications Vulkan, avec
  quelques bibliothèques utiles.
---

# Environnement de développement

## Les langages

### C++

Tout d'abord, le langage que nous allons utiliser pour ce tutoriel est le C++. Il est possible de développer des applications Vulkan dans d’autre langage, si il existe le bindings pour. Cependant, il est généralement déconseillé d’utiliser d’autre langage de programmation. En effet, l’API Vulkan est développé en C et cela fera donc plus d'appels de fonction étrangère dans le C-land. Pour les langages qui sont interprétées \(à l'exception de Rust, par exemple\), la surcharge des appels de fonctions supplémentaires s'additionnera et compensera potentiellement les gains de performances que vous avez peut-être réalisés avec Vulkan en premier lieu.

### GLSL

GLSL pour OpenGL Shading Language, est un langage destiné à l'écriture des shaders qui n'est pas inconnu pour les développeurs OpenGL. De aprt sa simplicité, nous avons choisi de l'utilisé pour l'écriture des shaders de ce tutoriel.

{% hint style="info" %}
Nous partons du principe que vous avez déjà des connaissances en C++ et en GLSL
{% endhint %}

## Les librairies

### GLFW

GLFW est une bibliothèque multi-plateforme Open Source pour le développement OpenGL, OpenGL ES et Vulkan. Cette librairie fournit une API simple pour créer des fenêtres, des contextes et des surfaces, ainsi que recevoir des entrées de l'utilisateur. 

Vulkan ne possède pas d'outils pour créer des fenêtres \(ou surface comme nous le verrons par la suite\) car c'est une API indépendante de la plate-forme. Pour afficher notre rendu nous allons donc utiliser GLFW. Il existe évidenmment d'autres bibliothèques tels que SDL.

### GLSLang

Vulkan permet d’utiliser des shaders de tout type, à la condition qu’ils soient compilés dans le langage intermédiaire SPIR-V. Il existe plusieurs façon de compiler les shaders, dans ce tutoriel, nous allons utiliser l'éxécutable `glslangValidator` fourni par le projet [glslang](https://github.com/KhronosGroup/glslang) du KhronosGroup.

**Vidéo :**

{% embed url="https://youtu.be/5ViBDYaEz-g" %}

\*\*\*\*

