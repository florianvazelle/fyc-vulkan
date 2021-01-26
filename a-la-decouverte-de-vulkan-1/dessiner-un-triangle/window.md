# Window

Vulkan ignore la plateforme sur laquelle il opère et ne peut donc pas directement établir d'interface avec le gestionnaire de fenêtres. Nous verrons dans ce chapitre l'extension `VK_KHR_surface`. Nous pourrons ainsi obtenir l'objet `VkSurfaceKHR`, qui est un type abstrait de surface sur lequel nous pourrons effectuer des rendus. Cette surface sera en lien avec la fenêtre que nous avons créée grâce à GLFW.

L'extension `VK_KHR_surface`, qui se charge au niveau de l'instance, a déjà été ajoutée, car elle fait partie des extensions retournées par la fonction `glfwGetRequiredInstanceExtensions`.

La surface de fenêtre doit être créée juste après l'instance car elle peut influencer le choix du physical device. Il est important de noter que cette surface est complètement optionnelle, et vous pouvez l'ignorer si vous voulez effectuer du rendu off-screen ou du calcule, pour utiliser des compute shader par exemple.

## Création

Commencons par ajouter un nouveau membre à notre classe :

```cpp
VkSurfaceKHR surface;
```

La fonction `glfwCreateWindowSurface` va nous simplifier la vie, et implémente déjà la création de la surface correspondante à la bonne plateforme. Nous devons maintenant l'intégrer à notre programme. Ajoutez la fonction `createSurface` et appelez-la dans `initVulkan` après la création de l'instance et du DebugMessenger :

```cpp
void initVulkan() {
    createInstance();
    setupDebugMessenger();
    createSurface();
    pickPhysicalDevice();
    createLogicalDevice();
}

void createSurface() {
    if (glfwCreateWindowSurface(instance, window, nullptr, &surface) != VK_SUCCESS) {
        throw std::runtime_error("échec de la création de la window surface!");
    }
}
```

GLFW ne fournit aucune fonction pour détruire cette surface mais nous pouvons le faire nous-mêmes avec une simple fonction Vulkan :

```cpp
void cleanup() {
    ...
    vkDestroySurfaceKHR(instance, surface, nullptr);
    vkDestroyInstance(instance, nullptr);
    ...
}
```

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-7-window-surface.cpp" %}



