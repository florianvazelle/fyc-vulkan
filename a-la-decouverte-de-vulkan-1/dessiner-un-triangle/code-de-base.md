# Code de base

### Structure générale <a id="page_Structure-gnrale"></a>

Dans le chapitre précédent nous avons créé un projet Vulkan avec une configuration solide et nous l'avons testé. Nous recommençons ici à partir du code suivant :

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

### Intégrer GLFW <a id="page_Intgrer-GLFW"></a>

Vulkan marche très bien sans fenêtre si vous voulez l'utiliser pour du rendu sans écran \(offscreen rendering en Anglais\), mais c'est tout de même plus intéressant d'afficher quelque chose! Remplacez d'abord la ligne `#include <vulkan/vulkan.h>` par :

```cpp
#define GLFW_INCLUDE_VULKAN
#include <GLFW/glfw3.h>
```

 Le premier appel dans `initWindow` doit être `glfwInit()`, ce qui initialise la librairie. Dans la mesure où GLFW a été créée pour fonctionner avec OpenGL, nous devons lui demander de ne pas créer de contexte OpenGL avec l'appel suivant :

```cpp
glfwWindowHint(GLFW_CLIENT_API, GLFW_NO_API);
```

Dans la mesure où redimensionner une fenêtre n'est pas chose aisée avec Vulkan, nous verrons cela plus tard et l'interdisons pour l'instant.

```cpp
glfwWindowHint(GLFW_RESIZABLE, GLFW_FALSE);
```

 Il ne nous reste plus qu'à créer la fenêtre. Ajoutez un membre privé `GLFWWindow* m_window` pour en stocker une référence, et initialisez la ainsi :

```cpp
window = glfwCreateWindow(800, 600, "Vulkan", nullptr, nullptr);
```

Les trois premiers paramètres indiquent respectivement la largeur, la hauteur et le titre de la fenêtre. Le quatrième vous permet optionnellement de spécifier un moniteur sur lequel ouvrir la fenêtre, et le cinquième est spécifique à OpenGL.

Nous devrions plutôt utiliser des constantes pour la hauteur et la largeur dans la mesure où nous aurons besoin de ces valeurs dans le futur. J'ai donc ajouté ceci au-dessus de la définition de la classe `HelloTriangleApplication` :

```cpp
const uint32_t WIDTH = 800;
const uint32_t HEIGHT = 600;
```

et remplacé la création de la fenêtre par :

```cpp
window = glfwCreateWindow(WIDTH, HEIGHT, "Vulkan", nullptr, nullptr);
```

 Vous avez maintenant une fonction `initWindow` ressemblant à ceci :

```cpp
void initWindow() {
    glfwInit();

    glfwWindowHint(GLFW_CLIENT_API, GLFW_NO_API);
    glfwWindowHint(GLFW_RESIZABLE, GLFW_FALSE);

    window = glfwCreateWindow(WIDTH, HEIGHT, "Vulkan", nullptr, nullptr);
}
```

Pour s'assurer que l'application tourne jusqu'à ce qu'une erreur ou un clic sur la croix ne l'interrompe, nous devons écrire une petite boucle de gestion d'évènements :

```cpp
void mainLoop() {
    while (!glfwWindowShouldClose(window)) {
        glfwPollEvents();
    }
}
```

Ce code est relativement simple. GLFW récupère tous les évènements disponibles, puis vérifie qu'aucun d'entre eux ne correspond à une demande de fermeture de fenêtre. Ce sera aussi ici que nous appellerons la fonction qui affichera un triangle.

Une fois la requête pour la fermeture de la fenêtre récupérée, nous devons détruire toutes les ressources allouées et quitter GLFW. Voici notre première version de la fonction `cleanup` :

```cpp
void cleanup() {
    glfwDestroyWindow(window);

    glfwTerminate();
}
```

Si vous lancez l'application, vous devriez voir une fenêtre appelée "Vulkan" qui se ferme en cliquant sur la croix. Maintenant que nous avons une base pour notre application Vulkan, créons notre premier objet Vulkan!!

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-2-code-de-base.cpp" %}

{% embed url="https://youtu.be/INU4q5Nh0xU" %}

