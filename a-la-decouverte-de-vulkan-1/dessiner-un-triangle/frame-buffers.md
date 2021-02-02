# Frame Buffers

Nous avons beaucoup parlé de framebuffers dans les chapitres précédents, et nous avons mis en place la render pass pour qu'elle en accepte un du même format que les images de la swap chain. Pourtant nous n'en avons encore créé aucun.

Les attachements de différents types spécifiés durant la render pass sont liés en les considérant dans des objets de type `VkFramebuffer`. Un tel objet référence toutes les `VkImageView` utilisées comme attachements par une passe. Dans notre cas nous n'en aurons qu'un : un attachement de couleur, qui servira de cible d'affichage uniquement. Cependant l'image utilisée dépendra de l'image fournie par la swap chain lors de la requête pour l'affichage. Nous devons donc créer un framebuffer pour chacune des images de la swap chain et utiliser le bon au moment de l'affichage.

Pour cela créez un autre `std::vector` qui contiendra des framebuffers :

```cpp
std::vector<VkFramebuffer> swapChainFramebuffers;
```

 Nous allons remplir ce `vector` depuis une nouvelle fonction `createFramebuffers` que nous appellerons depuis `initVulkan` juste après la création de la pipeline graphique :

```cpp
void initVulkan() {
    ...
    createGraphicsPipeline();
    createFramebuffers();
}
```

Commencez par redimensionner le conteneur afin qu'il puisse stocker tous les framebuffers :

```cpp
void createFramebuffers() {
    swapChainFramebuffers.resize(swapChainImageViews.size());
}
```

Nous allons maintenant itérer à travers toutes les images et créer un framebuffer à partir de chacune d'entre elles :

```cpp
for (size_t i = 0; i < swapChainImageViews.size(); i++) {
    VkImageView attachments[] = {
        swapChainImageViews[i]
    };

    VkFramebufferCreateInfo framebufferInfo{};
    framebufferInfo.sType = VK_STRUCTURE_TYPE_FRAMEBUFFER_CREATE_INFO;
    framebufferInfo.renderPass = renderPass;
    framebufferInfo.attachmentCount = 1;
    framebufferInfo.pAttachments = attachments;
    framebufferInfo.width = swapChainExtent.width;
    framebufferInfo.height = swapChainExtent.height;
    framebufferInfo.layers = 1;

    if (vkCreateFramebuffer(device, &framebufferInfo, nullptr, &swapChainFramebuffers[i]) != VK_SUCCESS) {
        throw std::runtime_error("échec de la création d'un framebuffer!");
    }
}
```

 Pour la destruction, une simple boucle qui parcoures l'ensemble des frame, dans `cleanup`, et un appel a la fonction `vkDestroyFramebuffer` comme ce ci :

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

