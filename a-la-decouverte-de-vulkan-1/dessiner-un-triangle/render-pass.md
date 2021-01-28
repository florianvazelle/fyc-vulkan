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

## Dépendance

Les subpasses s'occupent automatiquement de la transition de l'organisation des images. Ces transitions sont contrôlées par des _subpass dependencies_. Elles indiquent la mémoire et l'exécution entre les subpasses. Nous n'avons certes qu'une seule subpasse pour le moment, mais les opérations avant et après cette subpasse comptent aussi comme des subpasses implicites.

Il existe deux dépendances préexistantes capables de gérer les transitions au début et à la fin de la render pass. Le problème est que cette première dépendance ne s'exécute pas au bon moment. Elle part du principe que la transition de l'organisation de l'image doit être réalisée au début de la pipeline, mais dans notre programme l'image n'est pas encore acquise à ce moment! Il existe deux manières de régler ce problème. Nous pourrions changer `waitStages` pour `imageAvailableSemaphore` à `VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT` pour être sûrs que la pipeline ne commence pas avant que l'image ne soit acquise, mais nous perdrions en performance car les shaders travaillant sur les vertices n'ont pas besoin de l'image. Il faudrait faire quelque chose de plus subtil. Nous allons donc plutôt faire attendre la render pass à l'étape `VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT` et faire la transition à ce moment. Cela nous donne de plus une bonne excuse pour s'intéresser au fonctionnement des subpass dependencies.

Celles-ci sont décrites dans une structure de type `VkSubpassDependency`. Créez en une dans la fonction `createRenderPass` :

```text
VkSubpassDependency dependency{};
dependency.srcSubpass = VK_SUBPASS_EXTERNAL;
dependency.dstSubpass = 0;
```

Les deux premiers champs permettent de fournir l'indice de la subpasse d'origine et de la subpasse d'arrivée. La valeur particulière `VK_SUBPASS_EXTERNAL` réfère à la subpass implicite soit avant soit après la render pass, selon que cette valeur est indiquée dans respectivement `srcSubpass` ou `dstSubpass`. L'indice `0` correspond à notre seule et unique subpasse. La valeur fournie à `dstSubpass` doit toujours être supérieure à `srcSubpass` car sinon une boucle infinie peut apparaître \(sauf si une des subpasse est `VK_SUBPASS_EXTERNAL`\).

```text
dependency.srcStageMask = VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT;
dependency.srcAccessMask = 0;
```

Les deux paramètres suivants indiquent les opérations à attendre et les étapes durant lesquelles les opérations à attendre doivent être considérées. Nous voulons attendre la fin de l'extraction de l'image avant d'y accéder, hors ceci est déjà configuré pour être synchronisé avec l'étape d'écriture sur l'attachement. C'est pourquoi nous n'avons qu'à attendre à cette étape.

```text
dependency.dstStageMask = VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT;
dependency.dstAccessMask = VK_ACCESS_COLOR_ATTACHMENT_WRITE_BIT;
```

Nous indiquons ici que les opérations qui doivent attendre pendant l'étape liée à l'attachement de couleur sont celles ayant trait à l'écriture. Ces paramètres permettent de faire attendre la transition jusqu'à ce qu'elle soit possible, ce qui correspond au moment où la passe accède à cet attachement puisqu'elle est elle-même configurée pour attendre ce moment.

```text
renderPassInfo.dependencyCount = 1;
renderPassInfo.pDependencies = &dependency;
```

Nous fournissons enfin à la structure ayant trait à la render pass un tableau de configurations pour les subpass dependencies.

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

