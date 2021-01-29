---
description: >-
  Le device physique est la carte graphique de notre ordinateur, alors que le
  device logique est l'interface entre l'application et notre composant.
---

# Device

## Physical

### Sélection d'un physical device <a id="page_Slection-d-un-physical-device"></a>

La librairie étant initialisée à travers `VkInstance`, nous pouvons dès à présent chercher et sélectionner une carte graphique \(physical device\) dans le système qui supporte les fonctionnalitées dont nous aurons besoin. Nous pouvons en fait en sélectionner autant que nous voulons et travailler avec chacune d'entre elles, mais nous n'en utiliserons qu'une dans ce tutoriel pour des raisons de simplicité.

Ajoutez la fonction `pickPhysicalDevice` et appelez la depuis `initVulkan` :

```cpp
void initVulkan() {
    createInstance();
    setupDebugMessenger();
    pickPhysicalDevice();
}

void pickPhysicalDevice() {

}
```

Nous stockerons le physical device que nous aurons sélectionnée dans un nouveau membre donnée de la classe, et celui-ci sera du type `VkPhysicalDevice`. Cette référence sera implicitement détruit avec l'instance, nous n'avons donc rien à ajouter à la fonction `cleanup`.

```cpp
VkPhysicalDevice physicalDevice = VK_NULL_HANDLE;
```

Lister les physical devices est un procédé très similaire à lister les extensions. Comme d'habitude, on commence par en lister le nombre.

```cpp
uint32_t deviceCount = 0;
vkEnumeratePhysicalDevices(instance, &deviceCount, nullptr);
```

Si aucun physical device ne supporte Vulkan, il est inutile de continuer l'exécution.

```cpp
if (deviceCount == 0) {
    throw std::runtime_error("aucune carte graphique ne supporte Vulkan!");
}
```

 Nous pouvons ensuite allouer un tableau contenant toutes les références aux [`VkPhysicalDevice`](https://www.khronos.org/registry/vulkan/specs/1.0/man/html/VkPhysicalDevice.html).

```cpp
std::vector<VkPhysicalDevice> devices(deviceCount);
vkEnumeratePhysicalDevices(instance, &deviceCount, devices.data());
```

Nous devons maintenant évaluer chacun des gpus et vérifier qu'ils conviennent pour ce que nous voudrons en faire, car toutes les cartes graphiques n'ont pas été crées égales. Voici une nouvelle fonction qui fera le travail de sélection :

```cpp
bool isDeviceSuitable(VkPhysicalDevice device) {
    return true;
}
```

Nous allons dans cette fonction vérifier que le physical device respecte nos conditions.

```cpp
for (const auto& device : devices) {
    if (isDeviceSuitable(device)) {
        physicalDevice = device;
        break;
    }
}

if (physicalDevice == VK_NULL_HANDLE) {
    throw std::runtime_error("aucun GPU ne peut exécuter ce programme!");
}
```

### Vérification des fonctionnalités de base <a id="page_Vrification-des-fonctionnalits-de-base"></a>

Pour évaluer la compatibilité d'un physical device nous devons d'abord nous informer sur ses capacités. Des propriétés basiques comme le nom, le type et les versions de Vulkan supportées peuvent être obtenues en appelant `vkGetPhysicalDeviceProperties`.

```cpp
VkPhysicalDeviceProperties deviceProperties;
vkGetPhysicalDeviceProperties(device, &deviceProperties);

VkPhysicalDeviceFeatures deviceFeatures;
vkGetPhysicalDeviceFeatures(device, &deviceFeatures);
```

Nous ne faisons que commencer donc nous prendrons la première carte supportant Vulkan :

```cpp
bool isDeviceSuitable(VkPhysicalDevice device) {
    return true;
}
```

### Familles de queues \(queue families\) <a id="page_Familles-de-queues-queue-families"></a>

Nous devons analyser quelles queue families existent sur le système et lesquelles correspondent aux commandes que nous souhaitons utiliser. Nous allons donc créer la fonction `findQueueFamilies` dans laquelle nous chercherons les commandes nous intéressant.

Nous allons chercher une queue qui supporte les commandes graphiques, la fonction pourrait ressembler à ça:

```cpp
uint32_t findQueueFamilies(VkPhysicalDevice device) {
    // Code servant à trouver la famille de queue "graphique"
}
```

Mais dans un des prochains chapitres, nous allons avoir besoin d'une autre famille de queues, il est donc plus intéressant de s'y préparer dès maintenant en empactant plusieurs indices dans une structure:

```cpp
struct QueueFamilyIndices {
    uint32_t graphicsFamily;
};

QueueFamilyIndices findQueueFamilies(VkPhysicalDevice device) {
    QueueFamilyIndices indices;
    // Code pour trouver les indices de familles à ajouter à la structure
    return indices
}
```

Ce n'est pas très pratique d'utiliser une valeur magique pour indiquer la non-existence d'une famille, comme n'importe quelle valeur de `uint32_t` peut théoriquement être une valeur valide d'index de famille, incluant `0`. Heureusement, le C++17 introduit un type qui permet la distinction entre le cas où la valeur existe et celui où elle n'existe pas:

```cpp
#include <optional>

...

std::optional<uint32_t> graphicsFamily;

std::cout << std::boolalpha << graphicsFamily.has_value() << std::endl; // faux

graphicsFamily = 0;

std::cout << std::boolalpha << graphicsFamily.has_value() << std::endl; // vrai
```

`std::optional` est un wrapper qui ne contient aucune valeur tant que vous ne lui en assignez pas une. Vous pouvez, quelque soit le moment, lui demander si il contient une valeur ou non en appelant sa fonction membre `has_value()`. On peut donc changer le code comme suit:

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

Récupérer la liste des queue families disponibles se fait de la même manière que d'habitude, avec la fonction `vkGetPhysicalDeviceQueueFamilyProperties` :

```cpp
uint32_t queueFamilyCount = 0;
vkGetPhysicalDeviceQueueFamilyProperties(device, &queueFamilyCount, nullptr);

std::vector<VkQueueFamilyProperties> queueFamilies(queueFamilyCount);
vkGetPhysicalDeviceQueueFamilyProperties(device, &queueFamilyCount, queueFamilies.data());
```

 La structure `VkQueueFamilyProperties` contient des informations sur la queue family, et en particulier le type d'opérations qu'elle supporte et le nombre de queues que l'on peut instancier à partir de cette famille. Nous devons trouver au moins une queue supportant `VK_QUEUE_GRAPHICS_BIT` :

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

Pour que ce soit plus pratique, nous allons aussi ajouter une fonction générique à la structure:

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

Bien, c'est tout ce dont nous aurons besoin pour choisir le bon physical device! La prochaine étape est de créer un logical device pour créer une interface avec la carte.

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

 Les prochaines informations à fournir sont les fonctionnalités du physical device que nous souhaitons utiliser. Ce sont celles dont nous avons vérifié la présence avec [`vkGetPhysicalDeviceFeatures`](https://www.khronos.org/registry/vulkan/specs/1.0/man/html/vkGetPhysicalDeviceFeatures.html) dans le chapitre précédent. Nous n'avons besoin de rien de spécial pour l'instant, nous pouvons donc nous contenter de créer la structure et de tout laisser à `VK_FALSE`, valeur par défaut. Nous reviendrons sur cette structure quand nous ferons des choses plus intéressantes avec Vulkan.

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

Avec le logical device et les queues nous allons maintenant pouvoir faire travailler la carte graphique! Dans le prochain chapitre nous mettrons en place les ressources nécessaires à la présentation des images à l'écran.

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-6-logical-device-et-queues \(1\).cpp" %}

{% embed url="https://youtu.be/jtrRpOznnIk" %}

