---
description: >-
  Le physical device/périphérique physique est la carte graphique de notre
  ordinateur, alors que le logical device/périphérique logique est l'interface
  entre l'application et notre composant.
---

# Devices

## Physical

### Sélection <a id="page_Slection-d-un-physical-device"></a>

Cette librairie déjà initialisée dans la partie `VkInstance`. Donc on peut maintenant choisir directement la carte graphique qui est compatible aux fonctionnalités de notre librairie. On peut aussi en sélectionner plusieurs et de travailler en même temps dessus. Mais dans le cas de ce tutoriel l'utilisation sera seulement sur une carte graphique, cela vas nous permettre de mieux comprendre.

On vas créer une fonction `pickPhysicalDevice` puis l'appelez dans la fonction `initVulkan` :

```cpp
void initVulkan() {
    createInstance();
    setupDebugMessenger();
    pickPhysicalDevice();
}

void pickPhysicalDevice() {

}
```

Nous allons créer une variable du type `VkPhysicalDevice` qui vas stocker notre Physical Device. Il sera automatiquement détruit, donc on n'a pas besoin de passer par la fonction `cleanup`.

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

### Queue Families <a id="page_Familles-de-queues-queue-families"></a>

Créent une fonction `findQueueFamilies` qui vas nous permettre de trouver les fonction que nous voulions utiliser.

```cpp
uint32_t findQueueFamilies(VkPhysicalDevice device) {
    ...
}
```

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

`std::optional` est un wrapper qui est vide si on lui assigne rien. On peut utiliser la fonction `has_value()` pour demander s'il est vide ou pas. Donc on vas modifier la fonction comme ainsi:

```cpp
#include <optional>

...

struct QueueFamilyIndices {
    std::optional<uint32_t> graphicsFamily;
    
    //Fonction Générique
    bool isComplete() {
        return graphicsFamily.has_value();
    }
};
```

 On peut maintenant commencer à implémenter `findQueueFamilies`:

```cpp
QueueFamilyIndices findQueueFamily(VkPhysicalDevice) {
    QueueFamilyIndices indices;

    //Recuperation de la liste des queue families disponibles
    uint32_t queueFamilyCount = 0;
    vkGetPhysicalDeviceQueueFamilyProperties(device, &queueFamilyCount, nullptr);
    
    // "VkQueueFamilyProperties" qui contient les info sur la queue family
    // comme le type d'operation mais aussi le nombre de queues qu'on peut créer.
    std::vector<VkQueueFamilyProperties> queueFamilies(queueFamilyCount);
    vkGetPhysicalDeviceQueueFamilyProperties(device, &queueFamilyCount, queueFamilies.data());
    
    //Trouvons aumoin une queue qui supporte "VK_QUEUE_GRAPHICS_BIT"
    int i = 0;
    for (const auto& queueFamily : queueFamilies) {
        if (queueFamily.queueFlags & VK_QUEUE_GRAPHICS_BIT) {
            indices.graphicsFamily = i;
        }
        
        //Sortir de la boucle des que c'est terminer
        if (indices.isComplete()) {
        break;
        }
    
        i++;
    }
    
    return indices;
}
```

Cette fois-ci modifions la fonction `isDeviceSuitable` pour qu'on vérifie que notre GPU reçois les information que nous avons envoyé. 

```cpp
bool isDeviceSuitable(VkPhysicalDevice device) {
    QueueFamilyIndices indices = findQueueFamilies(device);

    return indices.graphicsFamily.has_value();
}
```

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-5-physical-devices-et-queue-families.cpp" %}

{% embed url="https://youtu.be/AUV3yRiAwzg" %}

## Logical

Comme toujours ajoutons une variable de type `VkDevice` dont on aura besoin plus tard.

```cpp
VkDevice device;
```

On vas créer une fonction `createLogicalDevice`. Ce Logical Device vas nous servir comme interface, il se crée de la même manière qu'une Instance.

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

### Spécifier les Queues à créer <a id="page_Spcifier-les-fonctionnalits-utilises"></a>

Le Logical Device a aussi besoin des informations qu'il faut qu'on lui associes :

```cpp
void createLogicalDevice() {
    QueueFamilyIndices indices = findQueueFamilies(physicalDevice);
    
    //VkDeviceQueueCreateInfo indiquons le nombre de queues qu'on donne pour 
    //chaque family.On utilisera une queue pour l'instant pour simplifier.
    VkDeviceQueueCreateInfo queueCreateInfo{};
    queueCreateInfo.sType = VK_STRUCTURE_TYPE_DEVICE_QUEUE_CREATE_INFO;
    queueCreateInfo.queueFamilyIndex = indices.graphicsFamily.value();
    queueCreateInfo.queueCount = 1;
    
    //Pour l'instant on a une seule queue, mais il est toujours preferable de 
    //indiquer une priorité avec "pQueuePriorities" qui varie entre 0.0 et 1.0.
    float queuePriority = 1.0f;
    queueCreateInfo.pQueuePriorities = &queuePriority;
}
```

### Créer le Logical Device <a id="page_Crer-le-logical-device"></a>

Une fois terminer avec la structure Queue on peut maintenant passer à la structure principale qui s'appelle `VkDeviceCreateInfo`.

```cpp
void createLogicalDevice() {

    ...
    
    VkPhysicalDeviceFeatures deviceFeatures{};
    
    //Creation de la structure
    VkDeviceCreateInfo createInfo{};
    createInfo.sType = VK_STRUCTURE_TYPE_DEVICE_CREATE_INFO;
    
    createInfo.pQueueCreateInfos = &queueCreateInfo;
    createInfo.queueCreateInfoCount = 1;
    
    createInfo.pEnabledFeatures = &deviceFeatures;
    
    
    createInfo.enabledExtensionCount = 0;
    
    //Gere la compatibilités avec les anciennes implémentations
    if (enableValidationLayers) {
        createInfo.enabledLayerCount = static_cast<uint32_t>(validationLayers.size());
        createInfo.ppEnabledLayerNames = validationLayers.data();
    } else {
        createInfo.enabledLayerCount = 0;
    }
    
    //Controle d'erreur
    if (vkCreateDevice(physicalDevice, &createInfo, nullptr, &device) != VK_SUCCESS) {
        throw std::runtime_error("échec lors de la création d'un logical device!");
    }
}
```

Il ne faut pas oublier de le détruire avant le Physical Device :

```cpp
void cleanup() {
    vkDestroyDevice(device, nullptr);
    ...
}
```

### Récupérer des références aux queues <a id="page_Rcuprer-des-rfrences-aux-queues"></a>

Le type VkQueue va nous permete de stocker la référence de la Queue Family qui est compatible avec le graphisme. 

```cpp
vkGetDeviceQueue(device, indices.graphicsFamily.value(), 0, &graphicsQueue);
```

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-6-logical-device-et-queues \(1\).cpp" %}

{% embed url="https://youtu.be/QzLcD3rEJ-w" %}

