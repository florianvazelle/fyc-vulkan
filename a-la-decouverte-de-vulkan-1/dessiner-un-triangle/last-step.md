---
description: >-
  Finissons et admirons le triangle que nous, et surtout vous, avez réussi à
  afficher.
---

# La fin est proche

## Main Loop

Nous en avions déjà parlé, maintenant nous allons écrire notre fonction `drawFrame` qui sera exécuté à chaque frame pour en dessiner une.

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

## Synchronisation

Le fonction `drawFrame` réalisera les opérations suivantes :

* Acquérir une image depuis la Swap Chain
* Exécuter le Command Buffer correspondant au Frame Buffer dont l'attachment est l'image obtenue
* Retourner l'image à la Swap Chain pour présentation

Chacune de ces actions n'est réalisée qu'avec un appel de fonction. Cependant ce n'est pas aussi simple : les opérations sont par défaut exécutées de manière asynchrones. La fonction retourne aussitôt que les opérations sont lancées, et par conséquent l'ordre d'exécution est indéfini. Cela nous pose problème car chacune des opérations que nous voulons lancer dépendent des résultats de l'opération la précédant.

Il y a deux manières de synchroniser les évènements de la Swap Chain : les _fences_ et les _sémaphores_. Ces deux objets permettent d'attendre qu'une opération se termine en relayant un signal émis par un processus généré par la fonction à l'origine du lancement de l'opération.

Ils ont cependant une différence : l'état d'une fence peut être accédé depuis le programme à l'aide de fonctions telles que `vkWaitForFences` alors que les sémaphores ne le permettent pas. Les fences sont généralement utilisées pour synchroniser votre programme avec les opérations alors que les sémaphores synchronisent les opérations entre elles. Nous voulons synchroniser les queues, les commandes d'affichage et la présentation, donc les sémaphores nous conviennent le mieux.

## Sémaphores

Nous aurons besoin de deux sémaphores :

* un premier pour indiquer que l'acquisition de l'image s'est bien réalisée
* un second pour prévenir de la fin du rendu et permettre à l'image d'être retournée dans la Swap Chain

Créons deux membres données pour stocker ces sémaphores :

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

### Swap Chain

La toute première opération à réaliser dans `drawFrame` est d'acquérir une image depuis la Swap Chain. La Swap Chain étant une extension nous allons encore devoir utiliser des fonction suffixées de `KHR` :

```cpp
void drawFrame() {
    uint32_t imageIndex;
    vkAcquireNextImageKHR(device, swapChain, UINT64_MAX, imageAvailableSemaphore, VK_NULL_HANDLE, &imageIndex);
}
```

### Command Buffer

L'envoi à la queue et la synchronisation de celle-ci sont configurés à l'aide de paramètres dans la structure `VkSubmitInfo` que nous allons remplir.

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

La dernière étape pour l'affichage consiste à envoyer le résultat à la Swap Chain. La présentation est configurée avec une structure de type `VkPresentInfoKHR`, et nous ferons cela à la fin de la fonction `drawFrame`.

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

{% file src="../../.gitbook/assets/part-17-recreation-de-la-swap-chain.cpp" %}

{% embed url="https://youtu.be/3xSxc19OOEI" %}

**Quizz :**

Vous êtes arrivés à la fin du tutoriel, vous pouvez maintenant tester vos connaissance grâce au test final.

{% page-ref page="../../exercices/quizz-final.md" %}



