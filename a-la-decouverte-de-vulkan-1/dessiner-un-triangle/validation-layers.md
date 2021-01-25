# Débogage

Contrairement aux API graphiques traditionnelles, Vulkan distinct deux type de scénarios d'erreur possibles, Il y a :

* les **erreurs de "validité"**, qui sont des conditions d'erreur résultant d'une utilisation incorrecte de l'API graphique, c'est-à-dire que l'application ne respecte pas les règles d'utilisation de l'API qui sont requises pour obtenir un comportement bien défini des commandes émises. Ces règles sont décrites dans la spécification de toutes les commandes et structures de l'API dans des paragraphes intitulés "Valid Usage**"**.
* les **erreurs d'exécution**, qui sont des conditions d'erreur qui peuvent se produire même pendant l'exécution d'applications qui utilisent correctement l'API, comme par exemple un manque de mémoire. Les erreurs d'exécution sont des codes retournée suite a l'appelle de la méthode qui produit l'erreur. La spécification de Vulkan décrit tout les codes de retour possibles, que chaque commande peut renvoyer, dans des paragraphes intitulés "Return Codes**"**.

De nombreuses commandes de l'API Vulkan renvoient un code de résultat sous la forme de l'une des constantes de l'énumération `VkResult`, ces codes de résultat ne sont utilisés que pour indiquer des erreurs d'exécution et des informations d'état sur certaines opérations, mais ne signalent pas des informations sur le respect des conditions d'utilisation valides. 

En effet, cela permet aux applications de s'exécuter à des performances maximales, car les drivers n'ont alors pas à dépenser de temps d'exécution pour vérifier la bonne ou mauvaise utilisation potentielle des règles de spécification.

## Validation Layers

Vulkan est livré avec un ensemble de couches de validation et de débogage dans le cadre du SDK Vulkan. Lorsqu'un sous-ensemble de ces couches est activé, il s'insère automatiquement dans la chaîne d'appels de chaque appel d'API Vulkan émis par l'application pour effectuer son travail. Une description détaillée des différentes couches sort du cadre de cet article, mais les lecteurs curieux peuvent trouver plus d'informations ici.

L'avantage des couches de validation par rapport à l'approche adoptée par les API traditionnelles est que les applications ne doivent consacrer du temps à une vérification d'erreur approfondie que lorsque cela est explicitement demandé, pendant le développement et généralement lors de l'utilisation de versions de débogage de l'application.

Mieux encore, les couches de validation ne recherchent pas seulement les violations de l'utilisation autorisée de l'API, mais peuvent également signaler des avertissements concernant une utilisation incorrecte ou dangereuse potentielle de l'API, et sont même capables de signaler des avertissements de performance qui permettent aux développeurs d'identifier les endroits où l'API est utilisée correctement mais n'est pas utilisée de la manière la plus efficace.

### Mise en place

Nous allons maintenant préparer notre instance pour utilisé les couches de validation. 

Ajoutons une nouvelle fonction `CheckValidationLayerSupport`, qui devra vérifier si les couches que nous voulons utiliser sont disponibles. Liste-on d'abord les validation layers disponibles à l'aide de la fonction `vkEnumerateInstanceLayerProperties`. Elle s'utilise de la même façon que`vkEnumerateInstanceExtensionProperties`.

```cpp
bool Instance::CheckValidationLayerSupport() {
  uint32_t layerCount;
  vkEnumerateInstanceLayerProperties(&layerCount, nullptr);
  
  std::vector<VkLayerProperties> availableLayers(layerCount);
  vkEnumerateInstanceLayerProperties(&layerCount, availableLayers.data());
  
  for (const char* layerName : Instance::ValidationLayers) {
    bool layerFound = false;
    
    for (const auto& layerProperties : availableLayers) {
      if (strcmp(layerName, layerProperties.layerName) == 0) {
        layerFound = true;
        break;
      }
    }
    
    if (!layerFound) {
      return false;
    }
  }
  
  return true;
}
```

 Nous pouvons maintenant utiliser cette fonction lors de la création de notre instance.

```cpp
Instance::Instance(const char* appName, const char* engineName, bool validationLayers)
    : m_instance(VK_NULL_HANDLE), m_enableValidationLayers(validationLayers) {
  if (validationLayers && !CheckValidationLayerSupport()) {
    throw std::runtime_error("validation layers requested, but not available!");
  }
  ...
}
```

```cpp
VkInstanceCreateInfo createInfo{};
createInfo.sType = VK_STRUCTURE_TYPE_INSTANCE_CREATE_INFO;
createInfo.pApplicationInfo = &appInfo;

std::vector<const char*> extensions;
GetRequiredExtensions(extensions, validationLayers);
createInfo.enabledExtensionCount = static_cast<uint32_t>(extensions.size());
createInfo.ppEnabledExtensionNames = extensions.data();

VkDebugUtilsMessengerCreateInfoEXT debugCreateInfo;
if (validationLayers) {
  createInfo.enabledLayerCount = static_cast<uint32_t>(Instance::ValidationLayers.size());
  createInfo.ppEnabledLayerNames = Instance::ValidationLayers.data();

  DebugUtilsMessenger::PopulateDebugMessengerCreateInfo(debugCreateInfo);
  createInfo.pNext = (VkDebugUtilsMessengerCreateInfoEXT*)&debugCreateInfo;
} else {
  createInfo.enabledLayerCount = 0;
  createInfo.pNext = nullptr;
}
```

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-4-validation-layers.cpp" %}



