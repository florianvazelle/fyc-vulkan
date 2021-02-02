---
description: >-
  La Render Pass va permettre d'indiquer combien chaque FrameBuffer aura de
  buffer de couleur, de profondeur, combien de sample... et comment les
  utiliser.
---

# Render Pass

Une Render Pass \(passe de rendu en francais\) est une description générale des étapes de nos commandes de rendu, que nous verrons par la suite, ces étapes sont divisées en ressources utilisées pendant le rendu. Nous ne pouvons rien rendre dans Vulkan sans une passe de rendu. Et chaque passe de rendu doit comporter une ou plusieurs étapes. Ces étapes sont appelées, sous-passe.

## Description de l'attachment

La première chose que nous commençons à écrire est l'attachment de couleur. C'est plus ou moins une description de l'image qui sera utilisé par nos commandes de rendu. L'image utilisera le même format que la SwapChain.

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

Si on crée d'autre attachment le format peut être différent, il faut se référer à l'enum `VkFormat`. Mais par exemple, pour la couleur nous pourrions avoir la valeur `VK_FORMAT_R8G8B8A8_UNORM`, quand a la profondeur, on doit le déterminer en fonction du format supporter par le Device physique :

## Référence de attachment

```cpp
 const VkAttachmentReference colorRefSwapChain = {
      .attachment = 0,
      .layout     = VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL,
};
```

## Subpasses

Les Subpasses utilisent une collection de ressources définies pour la passe de rendu. Les ressources de la passe de rendu peuvent inclure des cibles de rendu \(couleur, profondeur, résolution\) et des données d'entrée \(ressources qui, potentiellement, étaient des cibles de rendu dans les sous-passes précédentes de la même passe de rendu\) et ces ressources s'appellent, attachment \(elles n'incluent pas les descripteurs, textures, samplers et buffers\).

Pourquoi ne les appelons-nous pas simplement des cibles de rendu ou des images ? Parce que nous ne les rendons pas seulement \(input attachment\) et parce que ce ne sont que des descriptions \(méta-données\). Les images qui doivent être utilisées comme attachments à l'intérieur des passes de rendu sont fournies via des buffers d'image.

Ici, restons simple et définissons une seul Subpass pour notre attachment reference :

```cpp
const VkSubpassDescription subpassSwapChain = {
      .pipelineBindPoint    = VK_PIPELINE_BIND_POINT_GRAPHICS,
      .colorAttachmentCount = 1,
      .pColorAttachments    = &colorRefSwapChain,
};
```

## Création du Render Pass

Maintenant que nous avons créé notre attachment et une Subpass, nous pouvons maintenant créer la Render Pass. Pour cela il faut créer une variable VkRenderPass renderPass et lui spécifier les attchments et les Subpasses. Ici, nous avons définis un vector pour chaque type :

```cpp
VkRenderPass renderPass;

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

Pour proprement détruire une Render Pass il suffit d'appeler la méthode :

```cpp
vkDestroyRenderPass(device, renderPass, nullptr);
```

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-12-render-pass.cpp" %}

{% embed url="https://youtu.be/lcpTghd0\_KQ" %}

{% file src="../../.gitbook/assets/part-13-finalisation-graphic-pipeline.cpp" %}

{% embed url="https://youtu.be/A8bq6AQLN3M" %}



