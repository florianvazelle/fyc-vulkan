---
description: La Swap Chain est une file d'attente d'images prêtes à être affichées.
---

# Swap Chain

Dans notre rendu 3D, nous allons devoir afficher plusieurs images à la suite pour animer un objet par exemple. Afin d'afficher un flux d'image continue, nous devons mettre en place une Swap Chain. 

Cette Swap Chain va servir à préparer les rendus d'images avant de les afficher à l'écran. Le nombre d'images dans la Swap Chain va dépendre des capacités de notre `VkSurfaceKHR.`

À noter également que toutes les cartes graphiques ne sont pas faites pour afficher des images à l'écran. Il est donc de la responsabilité du développeur de vérifier dans les extensions du device si la carte graphique peut gérer les Swap Chains.

Avant de créer la Swap Chain à proprement parlé, on doit récupérer certaines informations et contraintes que l'on désire. Le tout sera ensuite injecté dans la fonction de création de celle-ci. Parmi ces informations se trouve la surface,  le nombre d'images minimum, le format d'image, le format de couleur …

```cpp
VkSwapchainCreateInfoKHR createInfo = {
    .sType = VK_STRUCTURE_TYPE_SWAPCHAIN_CREATE_INFO_KHR,
    .surface = surface,
    .minImageCount = imageCount,
    .imageFormat = surfaceFormat.format,
    .imageColorSpace = surfaceFormat.colorSpace,
    .imageExtent = extent,
    .imageArrayLayers = 1,
    .imageUsage = VK_IMAGE_USAGE_COLOR_ATTACHMENT_BIT,
};
```

Une fois que la structure de paramètre `createInfo` est créée, on déclare une SwapChain avec le type suivant :

```cpp
VkSwapchainKHR swapChain; 
```

Et on l'initialise avec la fonction suivante \(on passe en paramètres les informations désirées ainsi que le Device concerné\) :

```cpp
if (vkCreateSwapchainKHR(device, &createInfo, nullptr, &swapChain) != VK_SUCCESS) {
    throw std::runtime_error("échec de la création de la swap chain!");
}
```

Afin de connaître le nombre d'images dans la Swap Chain on utilise la fonction :

```cpp
vkGetSwapchainImagesKHR(device, swapChain, &imageCount, nullptr);
```

Lorsque cette fonction est appelée, si le dernier paramètre vaut `nullptr`, la fonction retourne le nombre d'images que contient la Swap Chain. Ensuite, il faut appeler la fonction une nouvelle fois en passant en paramètre un pointeur vers un tableau de `VkImage`. Ce deuxième appel va remplir le tableau d'images avec les images de la Swap Chain.

On déclare le tableau de `VkImage` :

```cpp
std::vector<VkImage> swapChainImages;
```

Puis on appelle à nouveau la fonction `vkGetSwapchainImagesKHR()` :

```cpp
swapChainImages.resize(imageCount);
vkGetSwapchainImagesKHR(device, swapChain, &imageCount, swapChainImages.data());
```

Un bon réflexe à avoir est de garder dans une variable le nombre d'images contenu dans la Swap Chain ainsi que le format des images.

Grâce à la Swap Chain, nous avons à chaque instant une liste d'images prêtes à être affiché à l'écran ou sur lesquelles on peut travailler.

À la fin du programme, il est important de libérer la mémoire occupée par la Swap Chain. On peut libérer cette mémoire avec la fonction `vkDestroySwapchainKHR()`.

```cpp
void vkDestroySwapchainKHR(
    VkDevice                                    device,
    VkSwapchainKHR                              swapchain,
    const VkAllocationCallbacks*                pAllocator);
```

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-8-swap-chain.cpp" %}

{% embed url="https://youtu.be/3p-frgC12iQ" %}

{% file src="../../.gitbook/assets/part-9-image-views.cpp" %}

{% embed url="https://youtu.be/WTmmXXfsbBQ" %}

**Quizz :**

Vous êtes arrivés à la moitié du tutoriel, vous pouvez maintenant tester vos connaissances grâce au test intermédiaire.

{% page-ref page="../../exercices/quizz-intermediaire.md" %}



