---
description: >-
  Dans ce chapitre, nous allons voir les différents points pour correctement
  configurer son environnement pour développer des applications Vulkan, avec
  quelques bibliothèques utiles.
---

# Environnement de développement

## Le langage C++

Tout d'abord, le langage que nous allons utiliser pour ce tutoriel est le C++. Il est possible de développer des applications Vulkan dans d’autre langage, si il existe le bindings pour. Cependant, il est généralement déconseillé d’utiliser d’autre langage de programmation. En effet, l’API Vulkan est développé en C et cela fera donc plus d'appels de fonction étrangère dans le C-land. Pour les langues qui sont des langues interprétées \(à l'exception de Rust, par exemple\), la surcharge des appels de fonctions supplémentaires s'additionnera et compensera potentiellement les gains de performances que vous avez peut-être réalisés avec Vulkan en premier lieu.

{% hint style="info" %}
Nous partons du principe que vous avez déjà des connaissances en C++.
{% endhint %}

## Les librairies

### GLFW

Vulkan est une API indépendante de la plate-forme et n'inclut pas d'outils pour créer une fenêtre pour afficher les résultats rendus. Pour bénéficier des avantages multi-plateformes de Vulkan, nous allons utiliser la bibliothèque GLFW pour créer une fenêtre, qui prend en charge Windows, Linux et MacOS. Il existe d'autres bibliothèques disponibles à cet effet, comme SDL.

### GLM

Vulkan n'inclut pas non plus de bibliothèque pour les opérations d'algèbre linéaire. GLM est une bibliothèque conçue pour être utilisée avec les API graphiques et elles est également couramment utilisée avec OpenGL.

### GLSLang

Vulkan permet d’utiliser des shaders de tout type, à la condition qu’ils soient compilés dans le langage intermédiaire SPIR-V. Il existe plusieurs façon de compiler les shaders, dans ce tutoriel, nous allons utiliser la commande glslangValidator fourni par le projet glslang du KhronosGroup.

**Vidéo :**

{% embed url="https://youtu.be/5ViBDYaEz-g" %}

\*\*\*\*

