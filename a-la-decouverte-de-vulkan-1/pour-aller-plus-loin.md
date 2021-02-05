---
description: >-
  Nous avons pu, à travers ce tutoriel, voir déjà un certain nombre des concepts
  de Vulkan, mais celui ci nous en réserve encore un certain nombre.
---

# Pour aller plus loin

Un dernier concept pour la route, qui va nous permettre de décrire et d'envoyer nos propres données aux shaders \(Uniform Buffers, Sampler, Input Attachment ...\). Il s'agit des descriptors.

## Descriptor Layout

Pour commencer il faut créer un Descriptor Layout qui va permettre de fournir des informations sur chacun des descripteurs utilisés par les shaders lors de la création de la pipeline. Nous allons créer une fonction pour créer le Descriptor Layout.

```cpp
void initVulkan() {
    ...
    createDescriptorSetLayout();
    createGraphicsPipeline();
    ...
}

...

void createDescriptorSetLayout() {

}
```

Chaque `binding` doit être décrit à l'aide d'une structure de type `VkDescriptorSetLayoutBinding`.

```cpp
void createDescriptorSetLayout() {
    VkDescriptorSetLayoutBinding uboLayoutBinding{};
    uboLayoutBinding.binding = 0;
    uboLayoutBinding.descriptorType = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
    uboLayoutBinding.descriptorCount = 1;
}
```

Les deux premiers champs permettent de fournir la valeur indiquée dans le shader avec `binding` et le type de descripteur auquel il correspond. Il est possible que la variable côté shader soit un tableau d'UBO, et dans ce cas il faut indiquer le nombre d'éléments qu'il contient dans le membre `descriptorCount`. 

```cpp
uboLayoutBinding.stageFlags = VK_SHADER_STAGE_VERTEX_BIT;
```

Nous devons aussi informer Vulkan des étapes shaders qui accéderont à cette ressource. Le champ de bits `stageFlags` permet de combiner toutes les étapes shader concernées. Nous mettons  `VK_SHADER_STAGE_VERTEX_BIT` qui indique que l'on envoie notre variable dans le vertex shader. Vous pouvez aussi fournir la valeur `VK_SHADER_STAGE_ALL_GRAPHICS` pour indiquer tout les types de shader.

Tous les `VkDescriptorSetLayoutBinding` ainsi créés, sont ensuite combinés en un seul objet `VkDescriptorSetLayout`. Créez pour cela un nouveau membre :

```cpp
VkDescriptorSetLayout descriptorSetLayout;
VkPipelineLayout pipelineLayout;
```

Nous pouvons créer cet objet à l'aide de la fonction `vkCreateDescriptorSetLayout`. Cette fonction prend en argument une structure de type `VkDescriptorSetLayoutCreateInfo`. Elle contient un tableau contenant nos `VkDescriptorSetLayoutBinding` :

```cpp
VkDescriptorSetLayoutCreateInfo layoutInfo{};
layoutInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_SET_LAYOUT_CREATE_INFO;
layoutInfo.bindingCount = 1;
layoutInfo.pBindings = &uboLayoutBinding;

if (vkCreateDescriptorSetLayout(device, &layoutInfo, nullptr, &descriptorSetLayout) != VK_SUCCESS) {
    throw std::runtime_error("echec de la creation d'un set de descripteurs!");
}
```

Nous devons fournir cette structure à Vulkan durant la création de la pipeline graphique. Ils sont transmis par la structure `VkPipelineLayoutCreateInfo`. Modifiez ainsi la création de cette structure :

```cpp
VkPipelineLayoutCreateInfo pipelineLayoutInfo{};
pipelineLayoutInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_LAYOUT_CREATE_INFO;
pipelineLayoutInfo.setLayoutCount = 1;
pipelineLayoutInfo.pSetLayouts = &descriptorSetLayout;
```

À notez que l'on peut spécifier une liste de `VkDescriptorSetLayout` alors qu'un seul peut inclure tous les `bindings` d'une pipeline. Nous y reviendrons, quand nous nous intéresserons aux pools de descripteurs.

Enfin, pour détruire notre nouvelle objet :

```cpp
void cleanup() {
    cleanupSwapChain();

    vkDestroyDescriptorSetLayout(device, descriptorSetLayout, nullptr);

    ...
}
```

## Descriptor Pool

Les Descriptor Set ne peuvent pas être crées directement. Il faut les allouer depuis une pool, comme les Command Buffers. Nous allons créer la fonction `createDescriptorPool` pour générer une pool de descripteurs.

```cpp
void initVulkan() {
    ...
    createDescriptorSetLayout();
    createGraphicsPipeline();
    createDescriptorPool();
    ...
}

...

void createDescriptorPool() {

}
```

Nous devons d'abord indiquer les types de descripteurs et combien sont compris dans les sets. Nous utilisons pour cela une structure du type `VkDescriptorPoolSize` :

```cpp
VkDescriptorPoolSize poolSize{};
poolSize.type = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
poolSize.descriptorCount = static_cast<uint32_t>(swapChainImages.size());
```

Dans notre cas, c'est un descripteur par frame. Il n'est pas toujours nécessaire d'avoir autant de copie de Descriptor Set que d'images dans la Swap Chain. Si notre valeur a envoyer au shader est statique ou persistente durant plusieurs frames, nous pouvons simplement allouer un seul DescriptorSet, pour le renvoyer à chaque frame. Par contre si les valeurs du DescriptorSet sont misent à jour à chaque frame, il faut en avoir autant de fois que d'images dans la SwapChain. La raison est que le rendu n'est pas immédiat, on construit un Command Buffer qui va être soumis au GPU, mais il n'est pas garanti que la Command Buffer soit executée immédiatement. Du coup, si nous modifions une valeur référencée par un DescriptorSet soumis au GPU mais non éxecutée, nous remplaçons les données de la frame précédente par celle de la frame actuelle.

Maintenant, créons une structure `VkDescriptorPoolCreateInfo` et référençons notre `poolSize` : 

```cpp
VkDescriptorPoolCreateInfo poolInfo{};
poolInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_POOL_CREATE_INFO;
poolInfo.poolSizeCount = 1;
poolInfo.pPoolSizes = &poolSize;
```

Nous devons aussi spécifier le nombre maximum de Descriptor Set que nous sommes susceptibles d'allouer.

```cpp
poolInfo.maxSets = static_cast<uint32_t>(swapChainImages.size());
```

Ensuite on crée le nouveau membre `descriptorPool`, puis appelons `vkCreateDescriptorPool` pour l'initialiser :

```cpp
VkDescriptorPool descriptorPool;

...

void createDescriptorPool() {
    ...

    if (vkCreateDescriptorPool(device, &poolInfo, nullptr, &descriptorPool) != VK_SUCCESS) {
        throw std::runtime_error("echec de la creation de la pool de descripteurs!");
    }
}
```

La pool doit être recréée en même temps que la Swap Chain. On va donc la détruire dans `cleanupSwapChain`.

```cpp
void cleanupSwapChain() {
    ...
    for (size_t i = 0; i < swapChainImages.size(); i++) {
        vkDestroyBuffer(device, uniformBuffers[i], nullptr);
        vkFreeMemory(device, uniformBuffersMemory[i], nullptr);
    }
    
    vkDestroyDescriptorPool(device, descriptorPool, nullptr);

    ...
}
```

Et recréée dans `recreateSwapChain` :

```cpp
void recreateSwapChain() {
    ...
    createDescriptorPool();
    createCommandBuffers();
}
```

## Descriptor Set

Nous pouvons maintenant allouer les Descriptor Set. Créons pour cela la fonction `createDescriptorSets` :

```cpp
void initVulkan() {
    ...
    createDescriptorPool();
    createDescriptorSets();
    ...
}

...

void createDescriptorSets() {

}
```

L'allocation de cette ressource passe par la création d'une structure `VkDescriptorSetAllocateInfo`. Vous devez indiquer la Descriptor Pool, le Descriptor Layout et le nombre de sets à créer :

```cpp
std::vector<VkDescriptorSetLayout> layouts(swapChainImages.size(), descriptorSetLayout);
VkDescriptorSetAllocateInfo allocInfo{};
allocInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_SET_ALLOCATE_INFO;
allocInfo.descriptorPool = descriptorPool;
allocInfo.descriptorSetCount = static_cast<uint32_t>(swapChainImages.size());
allocInfo.pSetLayouts = layouts.data();
```

Créez le nouveau membre `descriptorPool`, ci dessous, et allouez les Descriptor Set avec `vkAllocateDescriptorSets` :

```cpp
VkDescriptorPool descriptorPool;
std::vector<VkDescriptorSet> descriptorSets;

...


void createDescriptorSets() {
    ...
    descriptorSets.resize(swapChainImages.size());
    if (vkAllocateDescriptorSets(device, &allocInfo, descriptorSets.data()) != VK_SUCCESS) {
        throw std::runtime_error("echec de l'allocation d'un set de descripteurs!");
    }
}
```

Les descripteurs référant à un buffer doivent être configurés avec une structure `VkDescriptorBufferInfo`. Elle indique le buffer contenant les données, et où les données sont stockées.

```cpp
for (size_t i = 0; i < swapChainImages.size(); i++) {
    VkDescriptorBufferInfo bufferInfo{};
    bufferInfo.buffer = uniformBuffers[i];
    bufferInfo.offset = 0;
    bufferInfo.range = sizeof(UniformBufferObject);
}
```

La configuration des Descriptor Set sont maintenant possibles, nous allons les mettre à jour à l'aide de la fonction `vkUpdateDescriptorSets`. Elle prend un tableau de `VkWriteDescriptorSet` en paramètres. Les deux premiers champs spécifient le set à mettre à jour et l'indice du binding auquel il correspond.

```cpp
VkWriteDescriptorSet descriptorWrite{};
descriptorWrite.sType = VK_STRUCTURE_TYPE_WRITE_DESCRIPTOR_SET;
descriptorWrite.dstSet = descriptorSets[i];
descriptorWrite.dstBinding = 0;
```

Nous devons encore indiquer le type du descripteur. Il est possible de mettre à jour plusieurs descripteurs d'un même type en même temps. 

```cpp
descriptorWrite.descriptorType = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
descriptorWrite.descriptorCount = 1;
```

Enfin, pour indiquer la valeur du descripteur, il faut compléter le champs  `pBufferInfo` ou `pImageInfo` pour les descripteurs liés aux images.

```cpp
descriptorWrite.pBufferInfo = &bufferInfo;
```

Enfin, nous mettons à jour nos Descriptor Set :

```cpp
vkUpdateDescriptorSets(device, 1, &descriptorWrite, 0, nullptr);
```

