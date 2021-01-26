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

Mais ajoutons d'abord deux variables spécifiant les layers à activer et si le programme doit en effet les activer. J'ai choisi d'effectuer ce choix selon si le programme est compilé en mode debug ou non. La macro `NDEBUG` fait partie du standard c++ et correspond au second cas.

```cpp
const uint32_t WIDTH = 800;
const uint32_t HEIGHT = 600;

const std::vector<const char*> validationLayers = {
    "VK_LAYER_KHRONOS_validation"
};

#ifdef NDEBUG
    constexpr bool enableValidationLayers = false;
#else
    constexpr bool enableValidationLayers = true;
#endif
```

Ajoutons une nouvelle fonction `CheckValidationLayerSupport`, qui devra vérifier si les couches que nous voulons utiliser sont disponibles. Liste-on d'abord les validation layers disponibles à l'aide de la fonction `vkEnumerateInstanceLayerProperties`. Elle s'utilise de la même façon que`vkEnumerateInstanceExtensionProperties`.

```cpp
bool checkValidationLayerSupport() {
    uint32_t layerCount;
    vkEnumerateInstanceLayerProperties(&layerCount, nullptr);

    std::vector<VkLayerProperties> availableLayers(layerCount);
    vkEnumerateInstanceLayerProperties(&layerCount, availableLayers.data());

    return false;
}
```

Vérifiez que toutes les layers de `validationLayers` sont présentes dans la liste des layers disponibles. Vous aurez besoin de `<cstring>` pour la fonction `strcmp`.

```cpp
for (const char* layerName : validationLayers) {
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
```

Nous pouvons maintenant utiliser cette fonction lors de la création de notre instance.

```cpp
void createInstance() {
    if (enableValidationLayers && !checkValidationLayerSupport()) {
        throw std::runtime_error("les validations layers sont activées mais ne sont pas disponibles!");
    }

    ...
}
```

Lancez maintenant le programme en mode debug et assurez-vous qu'il fonctionne. Si vous obtenez une erreur, référez-vous à la FAQ.

Modifions enfin la structure [`VkCreateInstanceInfo`](https://www.khronos.org/registry/vulkan/specs/1.0/man/html/VkCreateInstanceInfo.html) pour inclure les noms des validation layers à utiliser lorsqu'elles sont activées :

```cpp
if (enableValidationLayers) {
    createInfo.enabledLayerCount = static_cast<uint32_t>(validationLayers.size());
    createInfo.ppEnabledLayerNames = validationLayers.data();
} else {
    createInfo.enabledLayerCount = 0;
}
```

 Si l'appel à la fonction `checkValidationLayerSupport` est un succès, [`vkCreateInstance`](https://www.khronos.org/registry/vulkan/specs/1.0/man/html/vkCreateInstance.html) ne devrait jamais retourner `VK_ERROR_LAYER_NOT_PRESENT`, mais exécutez tout de même le programme pour être sûr que d'autres erreurs n'apparaissent pas.

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-4-validation-layers.cpp" %}



