---
description: >-
  Commençons par découvrir la structure générale de notre application et
  intégrons GLFW.
---

# Premiers pas

## Structure générale

Suite à la vidéo de création d'un environnement de développement Vulkan, nous pouvons commencer à rentrer dans le vif du sujet : le code. 

### HelloTriangleApplication

Pour cela, créons une base d'application Vulkan simple, en commençant par une classe `HelloTriangleApplication` qui contiendra toute les méthodes dont nous aurons besoin  :

```cpp
class HelloTriangleApplication {
public:
    void run() {
        initVulkan();
        mainLoop();
        cleanup();
    }

private:
    void initVulkan() {}

    void mainLoop() {}

    void cleanup() {}
};
```

### Main

Ensuite, notre méthode main qui démarrera notre application :

```cpp
int main() {
    HelloTriangleApplication app;

    try {
        app.run();
    } catch (const std::exception& e) {
        std::cerr << e.what() << std::endl;`
        return EXIT_FAILURE;
    }

    return EXIT_SUCCESS;
}
```

Nous reviendrons la-dessus, mais nous créerons beaucoup d'exceptions dans notre application et c'est pourquoi ajouter un `try / catch`  est plus confortable pour déboguer et voir toutes les erreurs que peut générer notre application.

## Intégration de GLFW

Il est possible de faire des applications Vulkan sans fenêtre, mais ce tutoriel à pour but d'afficher un triangle, donc gérons le coté graphique ! Pour ajouter, les lignes suivantes pour intégrer GLFW à votre application C++ :

```cpp
#define GLFW_INCLUDE_VULKAN
#include <GLFW/glfw3.h>
```

### Initialisation

Rentrons, à présent, dans la théorie du fonctionnement de Vulkan. Une des spécificités de Vulkan, comparé à OpenGL, est que l'on doit lui spécifier des extensions qui sont tout simplement des fonctionnalités supplémentaires.

Nous allons voir cela plus précisément dans le chapitre suivant, consacré aux instances, mais c'est extensions ont besoin d’être déclaré avant la création de l'instance. Et l'extension dont nous avons besoin ici, est celle de GLFW. 

{% hint style="warning" %}
Il est important d'appeler `glfwInit()` avant la création de l'instance. Sinon nous ne pourrons tout simplement pas utiliser GLFW avec Vulkan.
{% endhint %}

```cpp
glfwInit();
```

GLFW a initialement été créé pour fonctionner avec OpenGL, comme le spécifie la documentation officielle de GLFW \([GLFW - Vulkan guide](https://www.glfw.org/docs/latest/vulkan_guide.html)\). À moins d'utiliser OpenGL ou OpenGL ES avec la même fenêtre que Vulkan, il n'est pas nécessaire de créer un contexte. Nous pouvons donc le désactiver avec l'indicateur `GLFW_CLIENT_API`.

```cpp
glfwWindowHint(GLFW_CLIENT_API, GLFW_NO_API);
```

Redimensionner une fenêtre n'est pas aussi simple qu'avec OpenGL et nous n'aborderons pas ce point dans ce tutoriel. Nous pouvons donc le désactiver avec l'indicateur `GLFW_RESIZABLE`.

```cpp
glfwWindowHint(GLFW_RESIZABLE, GLFW_FALSE);
```

### Création

Pour créer la fenêtre, ajoutons un attribut `GLFWWindow* m_window`  à notre classe `HelloTriangleApplication` et initialisons la ainsi :

```cpp
const uint32_t WIDTH = 800;
const uint32_t HEIGHT = 600;

m_window = glfwCreateWindow(WIDTH, HEIGHT, "Vulkan", nullptr, nullptr);
```

### Boucle d'événements

Maintenant ajoutons dans notre méthode `mainLoop`une boucle de gestion d’évènements, un classique de tout programme utilisant GLFW, qui va nous permettre de faire tourner notre application jusqu'à ce qu'une erreur ou autre évènement ne l'interrompe :

```cpp
void mainLoop() {
    while (!glfwWindowShouldClose(window)) {
        // On placera ici la fonction de dessin
        glfwPollEvents();
    }
}
```

### Destruction

Lors de la fermeture de notre programme, il faut :

* détruire toutes les ressources allouées avec `glfwDestroyWindow`,
* quitter GLFW, avec `glfwTerminate`

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-2-code-de-base.cpp" %}

{% embed url="https://youtu.be/INU4q5Nh0xU" %}

