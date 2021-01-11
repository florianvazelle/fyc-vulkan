# Render Pass

Un render pass \(passe de rendu en francais\) est une description générale des étapes de nos commandes de dessin, que nous verrons par la suite, ces étapes sont divisées en ressources utilisées pendant le rendu. Nous ne pouvons rien rendre dans Vulkan sans une passe de rendu. Et chaque passe de rendu doit comporter une ou plusieurs étapes. Ces étapes sont appelées, sous-passe.

Une render pass va surtout servir a indiquer combien chaque framebuffer aura de buffers de couleur et de profondeur, combien de samples il faudra utiliser avec chaque frambuffer et comment les utiliser tout au long des opérations de rendu.

## Description de l'attachment

```cpp
const VkAttachmentDescription colorAttachmentSwapChain = {
      // Le format doit correspondre au format de la chaîne d'échange
      .format         = m_swapChain.imageFormat(),
      .samples        = VK_SAMPLE_COUNT_1_BIT,        // Pas de multisampling
      .loadOp         = VK_ATTACHMENT_LOAD_OP_CLEAR,  // Effacer les données avant le rendu
      .storeOp        = VK_ATTACHMENT_STORE_OP_STORE, // puis stocker le résultat après
      .stencilLoadOp  = VK_ATTACHMENT_LOAD_OP_DONT_CARE,
      .stencilStoreOp = VK_ATTACHMENT_STORE_OP_DONT_CARE,
      .initialLayout  = VK_IMAGE_LAYOUT_UNDEFINED,
      // La mise en page finale doit être la même que la source de la présentation
      .finalLayout     = VK_IMAGE_LAYOUT_PRESENT_SRC_KHR,
};
```

Si on crée d'autre attachment le format peut être différent, il faut se référer à l'enum `VkFormat`. Mais par exemple, pour la couleur nous pourrions avoir la valeur `VK_FORMAT_R8G8B8A8_UNORM`, quand a la profondeur, on doit le déterminer en fonction du format supporter par le physical device :

```cpp

```

## Référence de attachment

```cpp
 const VkAttachmentReference colorRefSwapChain = {
      .attachment = 0,
      .layout     = VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL,
};
```

## Subpasses

Les sous-passes et chaque sous-passe utilisent une \(sous\) collection de ressources définies pour la passe de rendu. Les ressources de la passe de rendu peuvent inclure des cibles de rendu \(couleur, profondeur / gabarit, résolution\) et des données d'entrée \(ressources qui, potentiellement, étaient des cibles de rendu dans les sous-passes précédentes de la même passe de rendu\). Et ces ressources s'appellent, attachment \(elles n'incluent pas les descripteurs / textures / samplers et buffers\).

Pourquoi ne les appelons-nous pas simplement des cibles de rendu ou des images ? Parce que nous ne les rendons pas seulement \(input attachment\) et parce que ce ne sont que des descriptions \(méta-données\). Les images qui doivent être utilisées comme attachments à l'intérieur des passes de rendu sont fournies via des buffers d'image.

```cpp
const VkSubpassDescription subpassSwapChain = {
      .pipelineBindPoint = VK_PIPELINE_BIND_POINT_GRAPHICS,
      .colorAttachmentCount = 1,
      .pColorAttachments    = &colorRefSwapChain,
};
```

## Création du render pass

Maintenant que nous avons créé notre attachment et une subpass, nous pouvons maintenant créer la render pass. Pour cela il faut créer une nouvelle variable du type `VkRenderPass` :

```cpp
VkRenderPass renderPass;
```

L'objet représentant la render pass peut alors être créé en remplissant la structure `VkRenderPassCreateInfo` dans laquelle nous devons spécifier un tableau d'attachments et de subpasses.

```cpp
std::vector<VkAttachmentDescription> attachments = { colorAttachmentSwapChain };
std::vector<VkSubpassDescription> subpasses = { subpassSwapChain };

VkRenderPassCreateInfo renderPassInfo{};
renderPassInfo.sType = VK_STRUCTURE_TYPE_RENDER_PASS_CREATE_INFO;
renderPassInfo.attachmentCount = static_cast<uint32_t>(attachments.size());
renderPassInfo.pAttachments = attachments.data();
renderPassInfo.subpassCount = static_cast<uint32_t>(subpasses.size());
renderPassInfo.pSubpasses = subpasses.data();

if (vkCreateRenderPass(device, &renderPassInfo, nullptr, &renderPass) != VK_SUCCESS) {
    throw std::runtime_error("échec de la création de la render pass");
}
```

Pour proprement détruire une render pass il suffit d'appeler la méthode :

```cpp
vkDestroyRenderPass(device, renderPass, nullptr);
```

