# Frame Buffers

Dans le chapitre précédent, nous avons parlé de Frame Buffers, et nous avons mis en place la Render Pass pour qu'elle en accepte un du même format que les images de la Swap Chain. C'est le moment de découvrir cette notion pourtant très importante, complexe, mais très simple à mettre en oeuvre.

Chaque attachments que nous avons créé dans notre Render Pass doit être utilisé lorsque nous initialisons un Frame Buffer. De plus, nous initialiserons un Frame Buffer par `VkImageView` contenu dans la Swap Chain. 

Mais ce n'est pas obligatoire, par exemple, vous pouvez rendre une texture en créant votre propre `VkImage` avec les indicateurs d'utilisation `VK_IMAGE_USAGE_COLOR_ATTACHMENT_BIT | VK_IMAGE_USAGE_SAMPLED_BIT` afin qu'il puisse être écrit en tant qu'attachment de couleur, puis échantillonné dans un shader. Cependant, ici nous n'avons qu'un attachment de couleur, correspondant à celui de la Swap Chain.

## Création

Pour cela créez un `std::vector<VkFramebuffer> swapChainFramebuffers` et redimensionnons le  afin qu'il puisse stocker tous les Frame Buffers :

```cpp
void createFramebuffers() {
    swapChainFramebuffers.resize(swapChainImageViews.size());
}
```

Comme expliquer plus haut, parcourons toutes les images de la Swap Chain pour créer un Frame Buffer à partir de chacune d'entre elles :

```cpp
VkImageView attachments[1];

VkFramebufferCreateInfo framebufferInfo ={
    .sType           = VK_STRUCTURE_TYPE_FRAMEBUFFER_CREATE_INFO,
    .renderPass      = renderPass,
    .attachmentCount = 1,
    .pAttachments    = attachments,
    .width           = swapChainExtent.width,
    .height          = swapChainExtent.height,
    .layers          = 1,
};

for (size_t i = 0; i < swapChainImageViews.size(); i++) {
    attachments[0] = swapChainImageViews[i];    
    
    if (vkCreateFramebuffer(device, &framebufferInfo, nullptr, &swapChainFramebuffers[i]) != VK_SUCCESS) {
        throw std::runtime_error("échec de la création d'un framebuffer!");
    }
}
```

## Destruction

Pour la destruction, une simple boucle qui parcoure l'ensemble des frames, dans `cleanup`, et un appel a la fonction `vkDestroyFramebuffer` comme ce ci :

```cpp
void cleanup() {
    for (auto framebuffer : swapChainFramebuffers) {
        vkDestroyFramebuffer(device, framebuffer, nullptr);
    }

    ...
}
```

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-14-framebuffers.cpp" %}

{% embed url="https://youtu.be/PuwTlxupRW4" %}

\*\*\*\*

