---
description: La Swap Chain est une file d'attente d'images prêtes à être affichées.
---

# Swap Chain

Dans notre rendu 3D, nous allons devoir afficher plusieurs images à la suite pour animer un objet par exemple. Afin d'afficher un flux d'image continue, nous devons mettre en place une SwapChain. 

Cette SwapChain va servir à préparer les rendus d'image avant de les afficher à l'écran. Le nombre d'image dans la SwapChain va dépendre des capacités de notre `VkSurfaceKHR.`

A noter également que toute les cartes graphiques ne sont pas faite pour afficher des images à l'écran. Il est donc de la responsabilité du développeur de vérifier dans les extensions du device si la carte graphique peut gérer les SwapChain.

Avant de créer la SwapChain à proprement parlé, on doit récupérer certaines informations et contraintes que l'on désire. Le tout sera ensuite injecté dans la fonction de création de celle-ci. Parmi ces informations se trouve: la surface,  le nombre d'image minimum, le format d'image, le format de couleur …

```cpp
VkSwapchainCreateInfoKHR createInfo{};
    createInfo.sType = VK_STRUCTURE_TYPE_SWAPCHAIN_CREATE_INFO_KHR;
    createInfo.surface = surface;
    createInfo.minImageCount = imageCount;
    createInfo.imageFormat = surfaceFormat.format;
    createInfo.imageColorSpace = surfaceFormat.colorSpace;
    createInfo.imageExtent = extent;
    createInfo.imageArrayLayers = 1;
    createInfo.imageUsage = VK_IMAGE_USAGE_COLOR_ATTACHMENT_BIT;
```

Une fois que la structure de paramètre "createInfo" est crée, on déclare une SwapChain avec le type suivant:

```cpp
VkSwapchainKHR swapChain; 
```

Et on l'initialise avec la fonction suivante \(on passe en paramètre les informations désirée ainsi que le device concerné\):

```cpp
if (vkCreateSwapchainKHR(device, &createInfo, nullptr, &swapChain) != VK_SUCCESS) {
    throw std::runtime_error("échec de la création de la swap chain!");
}
```

Afin de connaitre le nombre d'image dans la SwapChain on utilise la fonction:

```cpp
vkGetSwapchainImagesKHR(device, swapChain, &imageCount, nullptr);
```

Lorsque cette fonction est appelée, si le dernier paramètre vaut "nullptr", la fonction retourne le nombre d'image que contient la SwapChain. Ensuite, il faut appeler la fonction une nouvelle fois en passant en paramètre un pointeur vers un tableau de "VkImage". Ce deuxième appel va remplir le tableau d'image avec les images de la SwapChain.

On déclare le tableau de "VkImage":

```cpp
std::vector<VkImage> swapChainImages;
```

Puis on appelle à nouveau la fonction "vkGetSwapchainImagesKHR\(\)":

```cpp
swapChainImages.resize(imageCount);
vkGetSwapchainImagesKHR(device, swapChain, &imageCount, swapChainImages.data());
```

Un bon reflexe à avoir est de garder dans une variable le nombre d'images contenu dans la SwapChain ainsi que le format des images.

Grâce à la SwapChain, nous avons à chaque instant une liste d'images prête à être affiché à l'écran ou sur lesquelles on peut travailler.





**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-8-swap-chain.cpp" %}

**Quizz :**

Vous êtes arrivés à la moitié du tutoriel, vous pouvez maintenant tester vos connaissance grâce au test intermédiaire.

{% page-ref page="../../exercices/quizz-intermediaire.md" %}



