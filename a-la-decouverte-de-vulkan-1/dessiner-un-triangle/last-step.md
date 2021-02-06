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

Nous voila avec ne application fonctionnelle d'où affichage d'un triangle dans une fenêtres. Mais il reste toujours des choses a améliorer. Dans ce projet on a prédéfini la taille de notre fenêtre et on a fait en sorte  de le verrouiller. Et si on voudrais redimensionner ou que la fenêtre sois adapter a notre écran comment ferait-on ? C'est dans cette dernière partie du chapitre que nous verrons la recréation de la swap chain pour que ca sois parfaitement adapter.

On vas commencer par créer la fonction nommer  `recreateSwapChain` qui vas réutiliser la fonction  `createSwapChain` et tout les autre fonction qui sont en relation avec celle-ci :

```cpp
void recreateSwapChain() {
    //Fonction prioritaire pour eviter les confllie 
    //des ressources qui est entrain d'etres utiliser.
    vkDeviceWaitIdle(device);  
                              
    createSwapChain();         //Logiquement la creation de la swapchain en 1er.
    createImageViews();        //2eme car image depend de al swapchain.
    createRenderPass();        //3eme depend du format des image.
    createGraphicsPipeline();  //
    createFramebuffers();      //        etc...
    createCommandBuffers();    //
}
```

Ajoutons une fonction  `cleanupSwapChain` pour êtres sure de l'avoir bien détruits avant de la recréer :

```cpp
void cleanupSwapChain() {
/* ---------------- Code deplacer de la fonction "cleanup" ---------------- */
    for (size_t i = 0; i < swapChainFramebuffers.size(); i++) {
        vkDestroyFramebuffer(device, swapChainFramebuffers[i], nullptr);
    }

    vkFreeCommandBuffers(device, commandPool, static_cast<uint32_t>(commandBuffers.size()), commandBuffers.data());

    vkDestroyPipeline(device, graphicsPipeline, nullptr);
    vkDestroyPipelineLayout(device, pipelineLayout, nullptr);
    vkDestroyRenderPass(device, renderPass, nullptr);

    for (size_t i = 0; i < swapChainImageViews.size(); i++) {
        vkDestroyImageView(device, swapChainImageViews[i], nullptr);
    }

    vkDestroySwapchainKHR(device, swapChain, nullptr);
/* ---------------- Code deplacer de la fonction "cleanup" ---------------- */
}
```

Voici a quoi ressemble la fonction `cleanup` après le déplacement du code. Il ne faut pas oublier de rajouter la fonction qu'on viens de créer `cleanupSwapChain` en premier pour question de redondance.

```cpp
void cleanup() {
    cleanupSwapChain();    //fonction creer juste avant

//  ------------------ Code restant ------------------
    for (size_t i = 0; i < MAX_FRAMES_IN_FLIGHT; i++) {
        vkDestroySemaphore(device, renderFinishedSemaphores[i], nullptr);
        vkDestroySemaphore(device, imageAvailableSemaphores[i], nullptr);
        vkDestroyFence(device, inFlightFences[i], nullptr);
    }

    vkDestroyCommandPool(device, commandPool, nullptr);

    vkDestroyDevice(device, nullptr);

    if (enableValidationLayers) {
        DestroyDebugReportCallbackEXT(instance, callback, nullptr);
    }

    vkDestroySurfaceKHR(instance, surface, nullptr);
    vkDestroyInstance(instance, nullptr);

    glfwDestroyWindow(window);

    glfwTerminate();
}
```

Maintenant nous devons changer la méthode  `chooseSwapExtent` , pour avoir une meilleure gestion du redimensionnement:

```cpp
VkExtent2D chooseSwapExtent(const VkSurfaceCapabilitiesKHR& capabilities) {
    if (capabilities.currentExtent.width != UINT32_MAX) {
        return capabilities.currentExtent;
    } else {
        int width, height;
        glfwGetFramebufferSize(window, &width, &height);

        VkExtent2D actualExtent = {
            static_cast<uint32_t>(width),
            static_cast<uint32_t>(height)
        };

        ...
    }
}
```

### Appel de la swap chain

Maintenant qu'on a les fonctions dont on a besoin il faut savoir ou les appeler. Tout simplement dans la fonction `drawFrame`juste après avoir reçus la prochaine image : 

```cpp
VkResult result = vkAcquireNextImageKHR(device, swapChain, UINT64_MAX, 
            imageAvailableSemaphores[currentFrame], VK_NULL_HANDLE, &imageIndex);
// VK_ERROR_OUT_OF_DATE_KHR permet de verifier 
// si la swap chain ne correpond plus au surface de la fenetre alors on la recreer.
if (result == VK_ERROR_OUT_OF_DATE_KHR) {
    recreateSwapChain();
    return;
} 
//VK_SUBOPTIMAL_KHR est la pour verifier si la taille de la fenetre 
//ne correspond plus a la swapchain, meme si la swapchain peut toujours etres utiliser.
else if (result != VK_SUCCESS && result != VK_SUBOPTIMAL_KHR) {
    throw std::runtime_error("échec de la présentation d'une image à la swap chain!");
}
```

### Explicitement gérer les redimensionnements

Ici on aura besoin d'une variable qui a pour but de garantir la redimensionnement de la fenêtre :

```cpp
bool framebufferResized = false;

...

void drawFrame() {

...
// vkQueuePresentKHR permet d'avoir une meilleur resultat avec sa valeurs retourner
result = vkQueuePresentKHR(presentQueue, &presentInfo);

if (result == VK_ERROR_OUT_OF_DATE_KHR || result == VK_SUBOPTIMAL_KHR || framebufferResized) {
    framebufferResized = false;
    recreateSwapChain();
} else if (result != VK_SUCCESS) {
    throw std::runtime_error("échec de la présentation d'une image!");
}

}
```

Il y a une fonction très utile qu'on vas utiliser ici c'est le  `glfwSetFrameBufferSizeCallback` qui vas nous permettre de savoir s'il y a eu une modification de la dimension de la fenêtre.

```cpp
void initWindow() {
    glfwInit();

    glfwWindowHint(GLFW_CLIENT_API, GLFW_NO_API);

    window = glfwCreateWindow(WIDTH, HEIGHT, "Vulkan", nullptr, nullptr);
    glfwSetWindowUserPointer(window, this);
    glfwSetFramebufferSizeCallback(window, framebufferResizeCallback);
}
static void framebufferResizeCallback(GLFWwindow* window, int width, int height) {

}
```

Comme vous le voyer nous avons créer une fonction static pour`glfwSetFramebufferSizeCallback`, car la librairie GLFZ ne sait pas utiliser les fonction d'une classe avec this.

```cpp
static void framebufferResizeCallback(GLFWwindow* window, int width, int height) {
    auto app = reinterpret_cast<HelloTriangleApplication*>(glfwGetWindowUserPointer(window));
    app->framebufferResized = true;
}
```

Vous pouvez maintenant tester votre application en le lançant et d'essayer de redimensionner la fenêtre.

### Gestion de la minimisation de la fenêtre <a id="page_Gestion-de-la-minimisation-de-la-fentre"></a>

Il reste une toute dernière chose a optimiser, c'est lorsque la fenêtre est minimiser, cela peut créer un problème a la swap chain. C'est ainsi nous utilisons une solution toute bête c'est a dire le mettre en pause si la fenêtre est minimiser et de recréer la swap chain une fois que la fenêtre sera revenu.

```cpp
void recreateSwapChain() {
    int width = 0, height = 0;
    glfwGetFramebufferSize(window, &width, &height);
    while (width == 0 || height == 0) {
        glfwGetFramebufferSize(window, &width, &height);
        glfwWaitEvents();
    }

    vkDeviceWaitIdle(device);

    ...
}
```

Je vous dit un grand bravo à tout ce qui on réussi a suivre jusqu'ici. Votre première application Vulkan est prête. Si vous voulez aller plus loin on vous a fournie plusieurs documentation et de site web sur Vulkan.

{% file src="../../.gitbook/assets/part-17-recreation-de-la-swap-chain.cpp" %}

{% embed url="https://youtu.be/3xSxc19OOEI" %}

**Quizz :**

Vous êtes arrivés à la fin du tutoriel, vous pouvez maintenant tester vos connaissance grâce au test final.

{% page-ref page="../../exercices/quizz-final.md" %}



