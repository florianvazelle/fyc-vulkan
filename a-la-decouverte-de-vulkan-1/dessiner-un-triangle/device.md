---
description: >-
  Le physical device/périphérique physique est la carte graphique de notre
  ordinateur, alors que le logical device/périphérique logique est l'interface
  entre l'application et notre composant.
---

# Devices

## Physical

### Sélection d'un physical device <a id="page_Slection-d-un-physical-device"></a>

Cette librairie déjà initialisée dans la partie `VkInstance`. Donc on peut maintenant choisir directement la carte graphique qui est compatible aux fonctionnalités de notre librairie. On peut aussi en sélectionner plusieurs et de travailler en même temps dessus. Mais dans le cas de ce tutoriel l'utilisation sera seulement sur une carte graphique, cela vas nous permettre de mieux comprendre.

On vas crée une fonction `pickPhysicalDevice` puis l'appelez dans la fonciton initVulkan :

```cpp
void initVulkan() {
    createInstance();
    setupDebugMessenger();
    pickPhysicalDevice();
}

void pickPhysicalDevice() {

}
```

Nous allons crée une variable du type `VkPhysicalDevice` qui vas stocker notre physical device. Il sera automatiquement détruit, donc on n'a pas besoin de passer par la fonction `cleanup`.

```cpp
void pickPhysicalDevice() {
    
    VkPhysicalDevice physicalDevice = VK_NULL_HANDLE;
    
    //Lister le nombre
    uint32_t deviceCount = 0;                    
    vkEnumeratePhysicalDevices(instance, &deviceCount, nullptr); 

    //Vérifie si la carte graphique est compatible avec Vulkan
    if (deviceCount == 0) {
    throw std::runtime_error("aucune carte graphique ne supporte Vulkan!");
    }
    
    //Allocation d'un tableau avec toutesles référence aux vkPhysicalDevice
    std::vector<VkPhysicalDevice> devices(deviceCount);
    vkEnumeratePhysicalDevices(instance, &deviceCount, devices.data());
    
    //Vérifie si le physical device est disponioble
    for (const auto& device : devices) {
        if (isDeviceSuitable(device)) {
            physicalDevice = device;
            break;
        }
    }
    
    if (physicalDevice == VK_NULL_HANDLE) {
        throw std::runtime_error("aucun GPU ne peut exécuter ce programme!");
    }
}

```

Nous évaluons les carte graphiques si il est compatible avec ce que nous voulons faire.

```cpp
bool isDeviceSuitable(VkPhysicalDevice device) {

    return true;
}
```

### Familles de queues \(queue families\) <a id="page_Familles-de-queues-queue-families"></a>

Créent une fonction `findQueueFamilies` qui vas nous permettre de trouver les fonction que nous voulions utiliser.

```cpp
uint32_t findQueueFamilies(VkPhysicalDevice device) {
    // Code servant à trouver la famille de queue "graphique"
}
```

Mais dans un des prochains chapitres, nous allons avoir besoin d'une autre famille de queues, il est donc plus intéressant de s'y préparer dès maintenant en regroupant plusieurs indices dans une structure :

Il existe une autre famille de queues que nous allons voir plus tard, mais on peut déjà commencer a le déclarer en rassemblant tout les indices dans la struct `QueueFamilyIndices`.

```cpp
struct QueueFamilyIndices {
    uint32_t graphicsFamily;
};

QueueFamilyIndices findQueueFamilies(VkPhysicalDevice device) {
    QueueFamilyIndices indices;

    return indices
}
```

`std::optional` est un wrapper qui ne contient aucune valeur tant que vous ne lui en assignez pas une. Vous pouvez, quelque soit le moment, lui demander s'il contient une valeur ou non en appelant sa fonction membre `has_value()`. On peut donc changer le code comme suit :

`std::optional` est un wrapper qui est vide si on lui assigne rien. On peut utiliser la fonction `has_value()` pour demander s'il est vide ou pas. Donc on vas modifier la fonction comme ainsi:

```cpp
#include <optional>

...

struct QueueFamilyIndices {
    std::optional<uint32_t> graphicsFamily;
};

QueueFamilyIndices findQueueFamilies(VkPhysicalDevice device) {
    QueueFamilyIndices indices;

    // Assigne l'index aux familles qui ont pu être trouvées

    return indices;
}
```

 On peut maintenant commencer à implémenter `findQueueFamilies`:

```cpp
QueueFamilyIndices findQueueFamily(VkPhysicalDevice) {
    QueueFamilyIndices indices;

    ...

    return indices;
}
```

Récupérer la liste des queue families disponibles se fait de la même manière que d'habitude avec la fonction `vkGetPhysicalDeviceQueueFamilyProperties` :

```cpp
uint32_t queueFamilyCount = 0;
vkGetPhysicalDeviceQueueFamilyProperties(device, &queueFamilyCount, nullptr);

std::vector<VkQueueFamilyProperties> queueFamilies(queueFamilyCount);
vkGetPhysicalDeviceQueueFamilyProperties(device, &queueFamilyCount, queueFamilies.data());
```

 La structure `VkQueueFamilyProperties` contient des informations sur la queue family et en particulier le type d'opération qu'elle supporte et le nombre de queues que l'on peut instancier à partir de cette famille. Nous devons trouver au moins une queue supportant `VK_QUEUE_GRAPHICS_BIT` :

```cpp
int i = 0;
for (const auto& queueFamily : queueFamilies) {
    if (queueFamily.queueFlags & VK_QUEUE_GRAPHICS_BIT) {
        indices.graphicsFamily = i;
    }

    i++;
}
```

Nous pouvons maintenant utiliser cette fonction dans `isDeviceSuitable` pour s'assurer que le physical device peut recevoir les commandes que nous voulons lui envoyer :

```cpp
bool isDeviceSuitable(VkPhysicalDevice device) {
    QueueFamilyIndices indices = findQueueFamilies(device);

    return indices.graphicsFamily.has_value();
}
```

Pour que ce soit plus pratique, nous allons aussi ajouter une fonction générique à la structure :

```cpp
struct QueueFamilyIndices {
    std::optional<uint32_t> graphicsFamily;

    bool isComplete() {
        return graphicsFamily.has_value();
    }
};

...

bool isDeviceSuitable(VkPhysicalDevice device) {
    QueueFamilyIndices indices = findQueueFamilies(device);

    return indices.isComplete();
}
```

 On peut également utiliser ceci pour sortir plus tôt de `findQueueFamilies`:

```cpp
for (const auto& queueFamily : queueFamilies) {
    ...

    if (indices.isComplete()) {
        break;
    }

    i++;
}
```

C'est tout ce dont nous aurons besoin pour choisir le bon physical device ! La prochaine étape est de créer un logical device pour créer une interface avec la carte.

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-5-physical-devices-et-queue-families.cpp" %}

{% embed url="https://youtu.be/AUV3yRiAwzg" %}

\*\*\*\*

## Logical

Commencez par ajouter un nouveau membre donnée pour stocker la référence au logical device.

```cpp
VkDevice device;
```

 Ajoutez ensuite une fonction `createLogicalDevice` et appelez-la depuis `initVulkan`.

```cpp
void initVulkan() {
    createInstance();
    setupDebugMessenger();
    pickPhysicalDevice();
    createLogicalDevice();
}

void createLogicalDevice() {

}
```

### Spécifier les queues à créer <a id="page_Spcifier-les-queues--crer"></a>

La création d'un logical device requiert encore que nous remplissions des informations dans des structures. La première de ces structures s'appelle [`VkDeviceQueueCreateInfo`](https://www.khronos.org/registry/vulkan/specs/1.0/man/html/VkDeviceQueueCreateInfo.html). Elle indique le nombre de queues que nous désirons pour chaque queue family. Pour le moment nous n'avons besoin que d'une queue originaire d'une unique queue family : la première avec un support pour les graphismes.

```cpp
QueueFamilyIndices indices = findQueueFamilies(physicalDevice);

VkDeviceQueueCreateInfo queueCreateInfo{};
queueCreateInfo.sType = VK_STRUCTURE_TYPE_DEVICE_QUEUE_CREATE_INFO;
queueCreateInfo.queueFamilyIndex = indices.graphicsFamily.value();
queueCreateInfo.queueCount = 1;

float queuePriority = 1.0f;
queueCreateInfo.pQueuePriorities = &queuePriority;
```

### Spécifier les fonctionnalités utilisées <a id="page_Spcifier-les-fonctionnalits-utilises"></a>

 Les prochaines informations à fournir sont les fonctionnalités du physical device que nous souhaitons utiliser. Ce sont celles dont nous avons vérifié la présence avec [`vkGetPhysicalDeviceFeatures`](https://www.khronos.org/registry/vulkan/specs/1.0/man/html/vkGetPhysicalDeviceFeatures.html) dans le chapitre précédent. Nous pouvons nous contenter de créer la structure et de tout laisser à `VK_FALSE`, valeur par défaut. Nous reviendrons sur cette structure quand nous ferons des choses plus élaborées avec Vulkan.

```cpp
VkPhysicalDeviceFeatures deviceFeatures{};
```

### Créer le logical device <a id="page_Crer-le-logical-device"></a>

Avec ces deux structure prêtes, nous pouvons enfin remplir la structure principale appelée [`VkDeviceCreateInfo`](https://www.khronos.org/registry/vulkan/specs/1.0/man/html/VkDeviceCreateInfo.html).

```cpp
VkDeviceCreateInfo createInfo{};
createInfo.sType = VK_STRUCTURE_TYPE_DEVICE_CREATE_INFO;

createInfo.pQueueCreateInfos = &queueCreateInfo;
createInfo.queueCreateInfoCount = 1;

createInfo.pEnabledFeatures = &deviceFeatures;

createInfo.enabledExtensionCount = 0;

if (enableValidationLayers) {
    createInfo.enabledLayerCount = static_cast<uint32_t>(validationLayers.size());
    createInfo.ppEnabledLayerNames = validationLayers.data();
} else {
    createInfo.enabledLayerCount = 0;
}

if (vkCreateDevice(physicalDevice, &createInfo, nullptr, &device) != VK_SUCCESS) {
    throw std::runtime_error("échec lors de la création d'un logical device!");
}
```

Les paramètres sont d'abord le physical device dont on souhaite extraire une interface, ensuite la structure contenant les informations, puis un pointeur optionnel pour l'allocation et enfin un pointeur sur la référence au logical device créé. Vérifions également si la création a été un succès ou non, comme lors de la création de l'instance.

Le logical device doit être explicitement détruit dans la fonction `cleanup` avant le physical device :

```cpp
void cleanup() {
    vkDestroyDevice(device, nullptr);
    ...
}
```

### Récupérer des références aux queues <a id="page_Rcuprer-des-rfrences-aux-queues"></a>

Les queue families sont automatiquement crées avec le logical device. Cependant nous n'avons aucune interface avec elles. Ajoutez un membre donnée pour stocker une référence à la queue family supportant les graphismes :

```cpp
VkQueue graphicsQueue;

vkGetDeviceQueue(device, indices.graphicsFamily.value(), 0, &graphicsQueue);
```

Avec le logical device et les queues nous allons maintenant pouvoir faire travailler la carte graphique ! Dans le prochain chapitre nous mettrons en place les ressources nécessaires à la présentation des images à l'écran.

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-6-logical-device-et-queues \(1\).cpp" %}

{% embed url="https://youtu.be/QzLcD3rEJ-w" %}

