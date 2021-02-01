---
description: >-
  Dans ce chapitre, nous allons voir les différents points pour correctement
  configurer son environnement afin de développer des applications Vulkan, avec
  quelques bibliothèques utiles.
---

# Environnement de développement

## Les langages

### C++

Tout d'abord, le langage que nous allons utiliser pour ce tutoriel est le C++. Il est possible de développer des applications Vulkan dans d’autres langages, s'il existe le bindings pour. Cependant, il est généralement déconseillé d’utiliser d’autres langages de programmation. En effet, l’API Vulkan est développé en C et cela fait donc plus d'appels de fonctions étrangères dans le C-land. Pour les langages qui sont interprétés \(à l'exception de Rust, par exemple\), la surcharge des appels de fonctions supplémentaires s'additionne et compense potentiellement les gains de performance que vous avez hypothétiquement réalisés avec Vulkan, en premier lieu.

Enfin, il est important de préciser qu'il existe une API Vulkan en C++ que l'on peut retrouver dans le répertoire [Vulkan-Hpp](https://github.com/KhronosGroup/Vulkan-Hpp). Cette API a pour objectif d'ajouter des fonctionnalités telles que la sécurité de type pour les énumérations et les champs de bits, la prise en charge des conteneurs STL.

{% hint style="info" %}
Pour rappel, la [sécurité de type](https://fr.wikipedia.org/wiki/S%C3%BBret%C3%A9_du_typage) signifie que le compilateur vous aidera à vérifier que vous ne mélangez pas les types de données.
{% endhint %}

 Cependant, nous allons utiliser l'API de Vulkan en C \(en incluant `vulkan/vulkan.h` et non `vulkan/vulkan.hpp`\). Ceci est un choix qui nous a semblé logique, toujours pour conserver l'optimisation que nous procure Vulkan, donc sans appel superflu de fonction.

### GLSL

GLSL pour OpenGL Shading Language est un langage destiné à l'écriture des shaders qui n'est pas inconnu pour les développeurs OpenGL. De part sa simplicité, nous avons choisi de l'utiliser pour l'écriture des shaders de ce tutoriel.

{% hint style="info" %}
Nous partons du principe que vous avez déjà des connaissances en C++ et en GLSL
{% endhint %}

## Les librairies

### GLFW

GLFW est une bibliothèque multiplateforme Open Source pour le développement OpenGL, OpenGL ES et Vulkan. Cette librairie fournit une API simple pour créer des fenêtres, des contextes et des surfaces, et aussi pour recevoir des entrées de l'utilisateur. 

Vulkan ne possède pas d'outils pour créer des fenêtres \(ou surface comme nous le verrons par la suite\) car c'est une API indépendante de la plateforme. Pour afficher notre rendu nous allons donc utiliser GLFW. Il existe évidemment d'autres bibliothèques telle que SDL.

### GLSLang

Vulkan permet d’utiliser des shaders de tout type, à la condition qu’ils soient compilés dans le langage intermédiaire SPIR-V. Il existe plusieurs façon de compiler les shaders, dans ce tutoriel, nous allons utiliser l'éxecutable `glslangValidator` fournit par le projet [glslang](https://github.com/KhronosGroup/glslang) du KhronosGroup.

**Vidéo :**

{% embed url="https://youtu.be/5ViBDYaEz-g" %}

\*\*\*\*

