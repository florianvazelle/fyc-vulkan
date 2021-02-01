---
description: La Swap Chain est une file d'attente d'images prêtes à être affichées.
---

# Swap Chain

Dans notre rendu 3D, nous allons devoir afficher plusieurs images à la suite pour animer un objet par exemple. Afin d'afficher un flux d'image continue, nous devons mettre en place une SwapChain. 

Cette SwapChain va servir à préparer les rendus d'image avant de les afficher à l'écran. Le nombre d'image dans la SwapChain va dépendre des capacités de notre `VkSurfaceKHR.`

Afin de connaitre le nombre d'image 

```text
void createSwapChain() { 

    SwapChainSupportDetails swapChainSupport = querySwapChainSupport(physicalDevice);
    
    
    VkSurfaceFormatKHR surfaceFormat = chooseSwapSurfaceFormat(swapChainSupport.formats);
    VkPresentModeKHR presentMode = chooseSwapPresentMode(swapChainSupport.presentModes);
    VkExtent2D extent = chooseSwapExtent(swapChainSupport.capabilities);
}
```

A noter également que toute les cartes graphiques ne sont pas faite pour afficher des images à l'écran. Il est donc de la responsabilité du développeur de vérifier dans les extensions du device.

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-8-swap-chain.cpp" %}

**Quizz :**

Vous êtes arrivés à la moitié du tutoriel, vous pouvez maintenant tester vos connaissance grâce au test intermédiaire.

{% page-ref page="../../exercices/quizz-intermediaire.md" %}



