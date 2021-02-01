---
description: L'instance permet de stocker l'état global de notre application.
---

# Instance

OpenGL fonctionne avec un système d'état global. L'état d'OpenGL est communément appelé le context. Lors de l'utilisation d'OpenGL, nous modifions souvent son état en définissant certaines options, en manipulant certains buffer puis en effectuant le rendu en utilisant le context actuel.

Avec Vulkan, il faut définir une instance, plus précisément un `VkInstance`. C'est un objet qui contient toutes les informations dont l'application Vulkan a besoin pour fonctionner.

## Création

Tout d'abord, nous allons créer un objet `VkInstance instance`. Pour ce ci, ajoutons le comme attribut privé, à notre classe représentant notre application. Puis initialisons le dans une méthode `createInstance` : 

```cpp
void createInstance() {
    VkApplicationInfo appInfo = {
        .sType              = VK_STRUCTURE_TYPE_APPLICATION_INFO,
        .pApplicationName   = "Hello Triangle",
        .applicationVersion = VK_MAKE_VERSION(1, 0, 0),
        .pEngineName        = "No Engine",
        .engineVersion      = VK_MAKE_VERSION(1, 0, 0),
        .apiVersion         = VK_API_VERSION_1_0,
    };
}
```

La structure, de type `VkApplicationInfo`, que l'on vient de définir permet de donner certaines informations optionnelle qui vont définir notre application.

Avec Vulkan, nous rencontrerons souvent des structures à remplir pour passer les créer/modifier des informations. Nous allons, maintenant, remplir le reste de la structure permettant la création de l'instance.

Elle permet d'informer le driver des extensions et des validations layers que nous utiliserons.

```cpp
uint32_t glfwExtensionCount = 0;
const char** glfwExtensions;

glfwExtensions = glfwGetRequiredInstanceExtensions(&glfwExtensionCount);

VkInstanceCreateInfo createInfo = {
    .sType                   = VK_STRUCTURE_TYPE_INSTANCE_CREATE_INFO,
    .pApplicationInfo        = &appInfo,
    .enabledExtensionCount   = glfwExtensionCount,
    .ppEnabledExtensionNames = glfwExtensions,
    .enabledLayerCount       = 0
};

if (vkCreateInstance(&createInfo, nullptr, &instance) != VK_SUCCESS) {
    throw std::runtime_error("Echec de la création de l'instance!");
}
```

## Destruction

C'est seulement à la fin du programme que nous détruirons notre object `VkInstance`, dans la méthode `cleanup`, comme ce ci :

```cpp
void cleanup() {
    vkDestroyInstance(instance, nullptr);
    
    glfwDestroyWindow(window);
    glfwTerminate();
}
```

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-3-instance.cpp" %}

{% embed url="https://youtu.be/AybxNdQ-hPk" %}

