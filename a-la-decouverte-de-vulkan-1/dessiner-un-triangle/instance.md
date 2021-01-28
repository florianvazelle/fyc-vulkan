# Instance

Tout d'abord, nous devons créer une instance de Vulkan. Un `VkInstance` est un objet qui contient toutes les informations dont l'application Vulkan a besoin pour fonctionner. Contrairement à OpenGL, Vulkan n'a pas d'état global. Pour cette raison, nous devons à la place stocker nos états dans cet objet. Dans ce chapitre, nous allons commencer un cours que nous utiliserons pour le reste du livre.

 Créez une fonction `createInstance` et appelez-la depuis la fonction `initVulkan` :

```cpp
void initVulkan() {
    createInstance();
}
```

Ajoutez ensuite un membre donnée représentant cette instance :

```cpp
private:
VkInstance instance;
```

Pour créer l'instance, nous allons d'abord remplir une première structure avec des informations sur notre application. Ces données sont optionnelles, mais elles peuvent fournir des informations utiles au driver pour optimiser ou diagnostiquer les erreurs lors de l'exécution, par exemple en reconnaissant le nom d'un moteur graphique. Cette structure s'appelle `VkApplicationInfo` :

```cpp
void initVulkan() {
    createInstance();
}

void createInstance() {
    VkApplicationInfo appInfo{};
    appInfo.sType = VK_STRUCTURE_TYPE_APPLICATION_INFO;
    appInfo.pApplicationName = "Hello Triangle";
    appInfo.applicationVersion = VK_MAKE_VERSION(1, 0, 0);
    appInfo.pEngineName = "No Engine";
    appInfo.engineVersion = VK_MAKE_VERSION(1, 0, 0);
    appInfo.apiVersion = VK_API_VERSION_1_0;
}
```

Comme mentionné précédemment, la plupart des structures Vulkan vous demandent d'expliciter leur propre type dans le membre `sType`. Cela permet d'indiquer la version exacte de la structure que nous voulons utiliser : il y aura dans le futur des extensions à celles-ci. Pour simplifier leur implémentation, les utiliser ne nécessitera que de changer le type `VK_STRUCTURE_TYPE_XXX` en `VK_STRUCTURE_TYPE_XXX_2` \(ou plus de 2\) et de fournir une structure complémentaire à l'aide du pointeur `pNext`. Nous n'utiliserons aucune extension, et donnerons donc toujours `nullptr` à `pNext`.

Avec Vulkan, nous rencontrerons souvent \(TRÈS souvent\) des structures à remplir pour passer les informations à Vulkan. Nous allons maintenant remplir le reste de la structure permettant la création de l'instance. Celle-ci n'est pas optionnelle. Elle permet d'informer le driver des extensions et des validation layers que nous utiliserons, et ceci de manière globale. Globale siginifie ici que ces données ne serons pas spécifiques à un périphérique. Nous verrons la signification de cela dans les chapitres suivants.

```cpp
VkInstanceCreateInfo createInfo{};
createInfo.sType = VK_STRUCTURE_TYPE_INSTANCE_CREATE_INFO;
createInfo.pApplicationInfo = &appInfo;

uint32_t glfwExtensionCount = 0;
const char** glfwExtensions;

glfwExtensions = glfwGetRequiredInstanceExtensions(&glfwExtensionCount);

createInfo.enabledExtensionCount = glfwExtensionCount;
createInfo.ppEnabledExtensionNames = glfwExtensions;

createInfo.enabledLayerCount = 0;

VkResult result = vkCreateInstance(&createInfo, nullptr, &instance);

if (vkCreateInstance(&createInfo, nullptr, &instance) != VK_SUCCESS) {
    throw std::runtime_error("Echec de la création de l'instance!");
}
```

### Vérification du support des extensions <a id="page_Vrification-du-support-des-extensions"></a>

Pour allouer un tableau contenant les détails des extensions nous devons déjà connaître le nombre de ces extensions. Vous pouvez ne demander que cette information en laissant le premier paramètre `nullptr` :

```cpp
uint32_t extensionCount = 0;
vkEnumerateInstanceExtensionProperties(nullptr, &extensionCount, nullptr);

std::vector<VkExtensionProperties> extensions(extensionCount);

vkEnumerateInstanceExtensionProperties(nullptr, &extensionCount, extensions.data());

std::cout << "Extensions disponibles :\n";

for (const auto& extension : extensions) {
    std::cout << '\t' << extension.extensionName << '\n';
}
```

### Libération des ressources <a id="page_Libration-des-ressources"></a>

L'instance contenue dans `VkInstance` ne doit être détruite qu'à la fin du programme. Nous la détruirons dans la fonction `cleanup` grâce à la fonction `vkDestroyInstance` :

```cpp
void cleanup() {
    vkDestroyInstance(instance, nullptr);

    glfwDestroyWindow(window);

    glfwTerminate();
}
```

Les paramètres de cette fonction sont évidents. Nous y retrouvons le paramètre pour un désallocateur que nous laissons nullptr. Toutes les ressources que nous allouerons à partir du prochain chapitre devront être libérées avant la libération de l'instance.

Avant d'avancer dans les notions plus complexes, créons un moyen de déboger notre programme avec les validations layers..

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-3-instance.cpp" %}

{% embed url="https://youtu.be/AybxNdQ-hPk" %}

