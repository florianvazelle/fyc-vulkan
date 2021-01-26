# Device

## Physical

### Sélection d'un physical device <a id="page_Slection-d-un-physical-device"></a>

La librairie étant initialisée à travers [`VkInstance`](https://www.khronos.org/registry/vulkan/specs/1.0/man/html/VkInstance.html), nous pouvons dès à présent chercher et sélectionner une carte graphique \(physical device\) dans le système qui supporte les fonctionnalitées dont nous aurons besoin. Nous pouvons en fait en sélectionner autant que nous voulons et travailler avec chacune d'entre elles, mais nous n'en utiliserons qu'une dans ce tutoriel pour des raisons de simplicité.

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

Nous stockerons le physical device que nous aurons sélectionnée dans un nouveau membre donnée de la classe, et celui-ci sera du type [`VkPhysicalDevice`](https://www.khronos.org/registry/vulkan/specs/1.0/man/html/VkPhysicalDevice.html). Cette référence sera implicitement détruit avec l'instance, nous n'avons donc rien à ajouter à la fonction `cleanup`.

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

**Vidéo / Code :**

## Logical



**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-5-physical-devices-et-queue-families.cpp" %}

