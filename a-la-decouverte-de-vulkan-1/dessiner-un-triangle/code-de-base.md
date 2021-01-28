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

Le premier appel dans `initWindow` doit être `glfwInit()`, ce dernier va initialiser la librairie. Dans la mesure où GLFW a été créée pour fonctionner avec OpenGL, nous devons lui demander de ne pas créer de contexte OpenGL avec l'appel suivant :

```cpp
glfwWindowHint(GLFW_CLIENT_API, GLFW_NO_API);
```

Redimensionner une fenêtre n'est pas aussi simple qu'en OpenGL, nous verrons cela plus tard et l'interdisons pour l'instant.

```cpp
glfwWindowHint(GLFW_RESIZABLE, GLFW_FALSE);
```

Il ne nous reste plus qu'à créer la fenêtre. Ajoutez un membre privé `GLFWWindow* m_window` pour en stocker une référence, et initialisez la ainsi :

```cpp
window = glfwCreateWindow(800, 600, "Vulkan", nullptr, nullptr);
```

Les trois premiers paramètres indiquent respectivement la largeur, la hauteur et le titre de la fenêtre. Le quatrième vous permet optionnellement de spécifier un moniteur sur lequel ouvrir la fenêtre, et le cinquième est spécifique à OpenGL.

Utilisons plutôt des constantes pour la hauteur et la largeur car nous aurons besoin de ces valeurs dans le futur. J'ai donc ajouté ceci au-dessus de la définition de la classe `HelloTriangleApplication` :

```cpp
const uint32_t WIDTH = 800;
const uint32_t HEIGHT = 600;
```

et remplacé la création de la fenêtre par :

```cpp
window = glfwCreateWindow(WIDTH, HEIGHT, "Vulkan", nullptr, nullptr);
```

Pour s'assurer que l'application tourne jusqu'à ce qu'une erreur ou autre ne l'interrompe, nous devons écrire une petite boucle de gestion d'évènements :

```cpp
void mainLoop() {
    while (!glfwWindowShouldClose(window)) {
        glfwPollEvents();
    }
}
```

Ce code est relativement simple et classique de tout programme utilisant GLFW. En effet, GLFW va récupèrer tous les évènements disponibles, puis vérifier qu'aucun d'entre eux ne correspond à une demande de fermeture de fenêtre. Ce sera aussi ici que nous appellerons la fonction qui affichera un triangle.

Nous devons aussi pensé à  détruire toutes les ressources allouées lors de la fermeture de notre programme et quitter GLFW. Ajoutez les deux méthodes correspondantes de GLFW à la fonction `cleanup` :

```cpp
void cleanup() {
    glfwDestroyWindow(window);

    glfwTerminate();
}
```

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-2-code-de-base.cpp" %}

{% embed url="https://youtu.be/INU4q5Nh0xU" %}

