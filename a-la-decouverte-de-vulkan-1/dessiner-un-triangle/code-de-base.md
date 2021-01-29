# Code de base

## Structure générale

Suite à la vidéo de création d'un environnement de développement Vulkan, nous pouvons commencer à rentrer dans le vif du sujet, le code. Pour cela, créons une base d'application Vulkan simple  :

```cpp
#include <vulkan/vulkan.h>

#include <iostream>
#include <stdexcept>
#include <functional>
#include <cstdlib>

class HelloTriangleApplication {
public:
    void run() {
        initVulkan();
        mainLoop();
        cleanup();
    }

private:
    void initVulkan() {

    }

    void mainLoop() {

    }

    void cleanup() {

    }
};

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

Nous définissons ici : 

* une classe `HelloTriangleApplication` qui contiendra toute les méthodes dont nous aurons besoin
* une méthode main qui démarrera notre application

## Intégration de GLFW

Il est possible de faire des applications Vulkan sans fenêtre, mais ce tutoriel à pour but d'afficher un triangle, donc gérons le coté graphique ! Pour ajouter, les lignes suivantes pour intégrer GLFW à votre application C++ :

```cpp
#define GLFW_INCLUDE_VULKAN
#include <GLFW/glfw3.h>
```

### Initialisation

Commencons maintenant à rentrer un peu dans la théorie du fonctionnement de Vulkan. Une des spécificité de Vulkan, comparé à OpenGL, est que l'on a besoin de lui spécifier des extensions qui sont tout simplement des fonctionnalités supplémentaires.

Nous allons voir cela plus précisément dans le chapitre suivant, consacré aux instances, mais c'est extensions ont besoin dêtre déclaré avant la création de l'instance. Et l'extension dont nous avons beoin ici, est celle de GLFW. Pour cela il est important d'appeler `glfwInit()` avant la création de l'instance. Sinon nous ne pourrons tout simplement pas utiliser GLFW avec Vulkan.

```cpp
glfwInit();
```

GLFW a initialement été créé pour fonctionner avec OpenGL. Comme le spécifie la documentation officiel de GLFW \([GLFW - Vulkan guide](https://www.glfw.org/docs/latest/vulkan_guide.html)\), à moins d'utilisier OpenGL ou OpenGL ES avec la même fenêtre que Vulkan, il n'est pas nécessaire de créer un contexte. Nous pouvons donc le désactiver avec l'indicateur `GLFW_CLIENT_API`.

```cpp
glfwWindowHint(GLFW_CLIENT_API, GLFW_NO_API);
```

Redimensionner une fenêtre n'est pas aussi simple qu'en OpenGL et nous ne verrons pas cela dans ce tutoriel. Nous pouvons donc le désactiver avec l'indicateur `GLFW_RESIZABLE`.

```cpp
glfwWindowHint(GLFW_RESIZABLE, GLFW_FALSE);
```

### Création

Pour créer la fenêtre, ajoutons un attribut `GLFWWindow* m_window` , à notre classe `HelloTriangleApplication`, et initialisons la ainsi :

```cpp
const uint32_t WIDTH = 800;
const uint32_t HEIGHT = 600;

m_window = glfwCreateWindow(WIDTH, HEIGHT, "Vulkan", nullptr, nullptr);
```

### Boucle d'événement 

Maintenant ajoutons dans notre méthode `mainLoop`, une boucle de gestion d’évènements, classique de tout programme utilisant GLFW, qui va nous permettre de faire tourner notre application jusqu'à ce qu'une erreur ou autre ne l'interrompe :

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

* détruire toutes les ressources allouées, avec `glfwDestroyWindow`,
* quitter GLFW, avec `glfwTerminate`

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-2-code-de-base.cpp" %}

{% embed url="https://youtu.be/INU4q5Nh0xU" %}

