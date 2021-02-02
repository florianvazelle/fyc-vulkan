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



\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* copyright \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

Vulkan ignore la plateforme sur laquelle il opère et ne peut donc pas directement établir d'interface avec le gestionnaire de fenêtres. Pour créer une interface permettant de présenter les rendus à l'écran, nous devons utiliser l'extension WSI \(Window System Integration\). Nous verrons dans ce chapitre l'extension `VK_KHR_surface`. Nous pourrons ainsi obtenir l'objet `VkSurfaceKHR`, qui est un type abstrait de surface sur lequel nous pourrons effectuer des rendus. Cette surface sera en lien avec la fenêtre que nous avons créée grâce à GLFW.

L'extension `VK_KHR_surface`, qui se charge au niveau de l'instance, a déjà été ajoutée, car elle fait partie des extensions retournées par la fonction `glfwGetRequiredInstanceExtensions`.

La surface de fenêtre doit être créée juste après l'instance car elle peut influencer le choix du physical device. Il est important de noter que cette surface est complètement optionnelle, et vous pouvez l'ignorer si vous voulez effectuer du rendu off-screen ou du calcul, pour utiliser des compute shaders, par exemple.

## Création

Commençons par ajouter un nouveau membre à notre classe :

```cpp
VkSurfaceKHR surface;
```

La fonction `glfwCreateWindowSurface` va nous simplifier la vie et implémente déjà la création de la surface correspondante à la bonne plateforme. Nous devons maintenant l'intégrer à notre programme. Ajoutez la fonction `createSurface` et appelez-la dans `initVulkan` après la création de l'instance et du DebugMessenger :

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

## Queue & Queue Family

### Demander le support pour la présentation

Bien que l'implémentation de Vulkan supporte le WSI, il est possible que d'autres éléments du système ne le supportent pas. Nous devons nous assurer que le logical device puisse présenter les rendus à la surface que nous avons créée. La présentation est spécifique aux queues families, ce qui signifie que nous devons trouver une queue family supportant cette présentation.

Il est possible que les queues families supportant les commandes d'affichage et celles supportant les commandes de présentation ne soient pas les mêmes, nous devons donc considérer que ces deux queues sont différentes. En fait, les spécificités des queue families diffèrent majoritairement entre les vendeurs, et assez peu entre les modèles d'une même série. Nous devons donc étendre la structure `QueueFamilyIndices` :

```cpp
struct QueueFamilyIndices {
    std::optional<uint32_t> graphicsFamily;
    std::optional<uint32_t> presentFamily;

    bool isComplete() {
        return graphicsFamily.has_value() && presentFamily.has_value();
    }
};
```

Nous devons ensuite modifier la fonction `findQueueFamilies` pour qu'elle cherche une queue family pouvant supporter les commandes de présentation. La fonction qui nous sera utile pour cela est `vkGetPhysicalDeviceSurfaceSupportKHR`. Elle possède quatre paramètres, le physical device, un indice de queue family, la surface et un booléen. Appelez-la depuis la même boucle que pour `VK_QUEUE_GRAPHICS_BIT` :

```cpp
VkBool32 presentSupport = false;
vkGetPhysicalDeviceSurfaceSupportKHR(device, i, surface, &presentSupport);
```

Vérifiez simplement la valeur du booléen et stockez la queue dans la structure si elle est intéressante :

```cpp
if (presentSupport) {
    indices.presentFamily = i;
}
```

### Création de la queue de présentation

Il nous reste à modifier la création du logical device pour extraire de celui-ci la référence à une presentation queue `VkQueue`. Ajoutez un membre donnée pour cette référence :

```cpp
VkQueue presentQueue;
```

Nous avons besoin de plusieurs structures `VkDeviceQueueCreateInfo`, une pour chaque queue family. Une manière de gérer ces structures est d'utiliser un `set` contenant tous les indices des queues et un `vector` pour les structures :

```cpp
#include <set>

...

QueueFamilyIndices indices = findQueueFamilies(physicalDevice);

std::vector<VkDeviceQueueCreateInfo> queueCreateInfos;
std::set<uint32_t> uniqueQueueFamilies = {indices.graphicsFamily.value(), indices.presentFamily.value()};

float queuePriority = 1.0f;
for (uint32_t queueFamily : uniqueQueueFamilies) {
    VkDeviceQueueCreateInfo queueCreateInfo{};
    queueCreateInfo.sType = VK_STRUCTURE_TYPE_DEVICE_QUEUE_CREATE_INFO;
    queueCreateInfo.queueFamilyIndex = queueFamily;
    queueCreateInfo.queueCount = 1;
    queueCreateInfo.pQueuePriorities = &queuePriority;
    queueCreateInfos.push_back(queueCreateInfo);
}
```

Puis modifiez `VkDeviceCreateInfo` pour qu'il pointe sur le contenu du vector :

```cpp
createInfo.queueCreateInfoCount = static_cast<uint32_t>(queueCreateInfos.size());
createInfo.pQueueCreateInfos = queueCreateInfos.data();
```

Si les queues sont les mêmes, nous n'avons besoin de les indiquer qu'une seule fois, ce dont le set s'assure. Ajoutez enfin un appel pour récupérer les queue families :

```cpp
vkGetDeviceQueue(device, indices.presentFamily.value(), 0, &presentQueue);
```

Si les queues sont les mêmes, les variables contenant les références contiennent la même valeur. Dans le prochain chapitre nous nous intéresserons à la swap chain, et verrons comment elle permet de présenter les rendus à l'écran.

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-7-window-surface.cpp" %}

{% embed url="https://youtu.be/jtrRpOznnIk" %}

