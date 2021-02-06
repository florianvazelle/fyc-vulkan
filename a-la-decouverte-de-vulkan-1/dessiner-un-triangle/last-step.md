---
description: >-
  Finissons, en écrivant notre fonction qui permettra d'assembler le tout et
  admirons le triangle que nous, et surtout vous, avez réussi à afficher.
---

# La fin est proche

## Synchronisation

Notre fonction doit réaliser trois étapes, pour assurer un rendu correct :

1. Récupérer une image de la Swap Chain
2. Exécuter le Command Buffer qui correspond au rendu du Frame Buffer précédemment créé \(pour rappelle celui ci nous permettra d'obtenir l'image\)
3. Retourner l'image à la Swap Chain pour affichage

Mais tout de suite nous allons parler de synchronisation. En effet, Vulkan exécute de manière asynchrone nos opérations \(comme Nodejs ou autre\). C'est au développeur que reviens le rôle d'expliciter à quelle moment nous devons attendre la réalisation d'une opération. Pour cela il existe deux évènements de synchronisation :

* les fences
* les sémaphores

Les fences vont principalement servir a synchroniser notre programme avec nos opérations. Quant au sémaphores, elles permettent de synchroniser nos opérations entre elles. Ce sont les sémaphores que nous allons utilisez ici.

## Sémaphores

Nous aurons besoin de deux sémaphores :

* un premier pour indiquer que l'acquisition de l'image s'est bien réalisée
* un second pour prévenir de la fin du rendu et permettre à l'image d'être retournée dans la Swap Chain

Créons deux variables :

```cpp
VkSemaphore imageAvailableSemaphore;
VkSemaphore renderFinishedSemaphore;
```

La création d'un sémaphore passe par le remplissage d'une structure de type `VkSemaphoreCreateInfo` :

```cpp
VkSemaphoreCreateInfo semaphoreInfo = {
    .sType = VK_STRUCTURE_TYPE_SEMAPHORE_CREATE_INFO,
};
```

Créons les sémaphores `vkCreateSemaphore` , tels que :

```cpp
if (vkCreateSemaphore(device, &semaphoreInfo, nullptr, &imageAvailableSemaphore) != VK_SUCCESS ||
    vkCreateSemaphore(device, &semaphoreInfo, nullptr, &renderFinishedSemaphore) != VK_SUCCESS) {

    throw std::runtime_error("échec de la création des sémaphores!");
}
```

Nous pourrons par la suite les détruire avec `vkDestroySemaphore`.

## Dessiner une Frame

Maintenant nous allons écrire notre fonction `drawFrame` .

```cpp
void mainLoop() {
    while (!glfwWindowShouldClose(window)) {
        glfwPollEvents();
        drawFrame();
    }
}

...

void drawFrame() {

}
```

Et pour écrire cette fonction, suivons donc les étapes explicité plus haut.

### Swap Chain

Pour commencer, récupèrons une image de la Swap Chain :

```cpp
uint32_t imageIndex;
vkAcquireNextImageKHR(device, swapChain, UINT64_MAX, imageAvailableSemaphore, VK_NULL_HANDLE, &imageIndex);
```

### Command Buffer

L'envoi à la queue et la synchronisation de celle-ci sont configurés à l'aide de paramètres dans la structure `VkSubmitInfo` :

```cpp
VkSemaphore waitSemaphores[] = {imageAvailableSemaphore};
VkPipelineStageFlags waitStages[] = {VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT};
VkSemaphore signalSemaphores[] = {renderFinishedSemaphore};

VkSubmitInfo submitInfo = {
    .sType                = VK_STRUCTURE_TYPE_SUBMIT_INFO,
    .waitSemaphoreCount   = 1,
    .pWaitSemaphores      = waitSemaphores,
    .pWaitDstStageMask    = waitStages,
    .commandBufferCount    = 1,
    .pCommandBuffers       = &commandBuffers[imageIndex],
    .signalSemaphoreCount = 1,
    .pSignalSemaphores    = signalSemaphores,
};

if (vkQueueSubmit(graphicsQueue, 1, &submitInfo, VK_NULL_HANDLE) != VK_SUCCESS) {
    throw std::runtime_error("échec de l'envoi d'un command buffer!");
}
```

### Présentation

Pour finir, affichons notre image. Pour cela il faut envoyer le résultat à la Swap Chain. Pour configurer la présentation, configurons une structure de type `VkPresentInfoKHR` :

```cpp
VkSwapchainKHR swapChains[] = {swapChain};

VkPresentInfoKHR presentInfo = {
    .sType              = VK_STRUCTURE_TYPE_PRESENT_INFO_KHR,
    .waitSemaphoreCount = 1,
    .pWaitSemaphores    = signalSemaphores,
    .swapchainCount     = 1,
    .pSwapchains        = swapChains,
    .pImageIndices      = &imageIndex,
};

vkQueuePresentKHR(presentQueue, &presentInfo); // émet la requête de présentation
```

## Résultat

![](https://vulkan-tutorial.com/images/triangle.png)

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-16-rendu-et-presentation.cpp" %}

{% embed url="https://youtu.be/xmKPSErwD6o" %}

## Recréation de la swap chain

{% file src="../../.gitbook/assets/part-17-recreation-de-la-swap-chain.cpp" %}

{% embed url="https://youtu.be/3xSxc19OOEI" %}

**Quizz :**

Vous êtes arrivés à la fin du tutoriel, vous pouvez maintenant tester vos connaissance grâce au test final.

{% page-ref page="../../exercices/quizz-final.md" %}



