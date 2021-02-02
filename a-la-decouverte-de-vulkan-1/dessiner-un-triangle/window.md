# Surfaces et Queues

Dans ce chapitre, nous allons voir comment afficher des résultats à l'écran et comment manipuler les composants d'affichage disponible.

Vulkan est avant tout conçu pour traiter des images et la plupart des applications issues de cette API auront pour objectif d'afficher des images de rendu.

Pour autant, la présentation d'image à l'écran n'est pas gérée par Vulkan directement \(On peut très bien utiliser l'API pour réaliser des rendus que l'on ne va pas afficher\). Nous allons donc devoir utiliser des extensions de Vulkan qui permettront la présentation d'image.

A noter que la présentation d'image peut être gérer de façon différente suivant les plateformes cibles.

Les objets qui permettent la présentation sont appelés **surface** et sont représenté dans Vulkan par des `VKSurfaceKHR`. On accède à ces objets avec l'extension VK\_KHR\_surface.

### Présentation sur Microsoft Windows

Avant de pouvoir présenter des images, nous devons nous assurer qu'au moins une queueFamily supporte les fonctions de présentation.

Sur Windows on peut appeler:

```cpp
VkBool32 vkGetPhysicalDeviceWin32PresentationSupportKHR(
    VkPhysicalDevice                            physicalDevice,
    uint32_t                                    queueFamilyIndex);
```

Cette fonction nous dit si la queueFamily supporte ou non la présentation.

Si au moins une queueFamily supporte la présentation, on peut créer notre surface:

```cpp
VkResult vkCreateWin32SurfaceKHR(
    VkInstance                                  instance,
    const VkWin32SurfaceCreateInfoKHR*          pCreateInfo,
    const VkAllocationCallbacks*                pAllocator,
    VkSurfaceKHR*                               pSurface);
```

On passe un pointeur vers une `VkSurfaceKHR*` et la fonction alloue l'objet pour nous \(On peut d'ailleurs utiliser un allocateur personnalisé\).

```cpp
typedef struct VkWin32SurfaceCreateInfoKHR {
    VkStructureType                 sType;
    const void*                     pNext;
    VkWin32SurfaceCreateFlagsKHR    flags;
    HINSTANCE                       hinstance;
    HWND                            hwnd;
} VkWin32SurfaceCreateInfoKHR;
```

On doit également passer en paramètre un pointeur vers un `VkWin32SurfaceCreateInfoKHR*` qui contiendra nos paramètres de de surface.

Par exemple pour une fenêtre **GLFW**:

```cpp
VkWin32SurfaceCreateInfoKHR createInfo{};
createInfo.sType = VK_STRUCTURE_TYPE_WIN32_SURFACE_CREATE_INFO_KHR;
createInfo.hwnd = glfwGetWin32Window(window);
createInfo.hinstance = GetModuleHandle(nullptr);
```

Lorsque l'on termine le programme il est important de libérer la mémoire allouée par la surface.

```cpp
vkDestroySurfaceKHR(instance, surface, nullptr);
```

### Création d'une Queue et traitement de l'affichage

Les `Queues` sont les composants du device qui vont effectuer les opérations. Chaque queue appartient à  une ou plusieurs `Queue families`. Les queues d'une même queue family sont identiques et ont les mêmes propriétés. En fonction du device que l'on utilise, leur nombre peut varier fortement.

Pour connaitre les propriétés de chaque queue family ainsi que leur nombre, on peut appeler la fonction vkGetPhysicalDeviceQueueFamilyProperties\(\).

```cpp
void vkGetPhysicalDeviceQueueFamilyProperties(
    VkPhysicalDevice                            physicalDevice,
    uint32_t*                                   pQueueFamilyPropertyCount,
    VkQueueFamilyProperties*                    pQueueFamilyProperties);
```

Afin de traiter nos opérations d'affichage on commence par créer une variable queue qui fera référence à notre queue d'affichage.

```cpp
VkQueue presentQueue;
```

Ensuite, on crée une structure qui va contenir les paramètres de création de la famille de queue qui nous intéresse \(la famille qui possède les attributs d'affichage\).

```cpp
VkDeviceQueueCreateInfo queueCreateInfo{};
    queueCreateInfo.sType = VK_STRUCTURE_TYPE_DEVICE_QUEUE_CREATE_INFO;
    queueCreateInfo.queueFamilyIndex = queueFamily;
    queueCreateInfo.queueCount = 1;
    queueCreateInfo.pQueuePriorities = &queuePriority;
```

Cette structure doit étre configurée à la création du device et passé en paramètre à la structure `VkDeviceCreateInfo{}`.

En effet, les queues et leur famille sont générée à la création du device. Ensuite on a juste à les récupérer avec la fonction `vkGetDeviceQueue()`.

```cpp
void vkGetDeviceQueue(
    VkDevice                                    device,
    uint32_t                                    queueFamilyIndex,
    uint32_t                                    queueIndex,
    VkQueue*                                    pQueue);
```

On passe en parametre notre pointeur vers notre "presentQueue" par exemple:

`vkGetDeviceQueue(device, indices.presentFamily.value(), 0, &presentQueue);`

Notre Queue de présentation est à présent prête à l'emploi, on verra dans un prochain chapitre comment lui faire exécuter des tâches à travers les `Command Buffer`.

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-7-window-surface.cpp" %}

{% embed url="https://youtu.be/jtrRpOznnIk" %}

