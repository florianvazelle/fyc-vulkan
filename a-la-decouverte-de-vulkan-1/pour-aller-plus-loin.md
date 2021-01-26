---
description: >-
  Nous avons pu, à travers ce tutoriel, voir déjà un certain nombre des concepts
  de Vulkan, mais celui ci nous en réserve encore un certain nombre.
---

# Pour aller plus loin

Un dernier concept pour la route, qui va nous permettre de décrire et d'envoyé nos propre données aux shader \(Uniform Buffers, Sampler, Input Attachment ...\). Il s'agit des descriptor.

## DescriptorLayout

Pour commencer il faut créer un descriptor layout qui va permettre de fournir des informations sur chacun des descripteurs utilisés par les shaders lors de la création de la pipeline. Nous allons créer une fonction pour gérer toute cette information, et ainsi pour créer le set de descripteurs.

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

Les deux premiers champs permettent de fournir la valeur indiquée dans le shader avec `binding` et le type de descripteur auquel il correspond. Il est possible que la variable côté shader soit un tableau d'UBO, et dans ce cas il faut indiquer le nombre d'éléments qu'il contient dans le membre `descriptorCount`. Cette possibilité pourrait être utilisée pour transmettre d'un coup toutes les transformations spécifiques aux différents éléments d'une structure hiérarchique. Nous n'utilisons pas cette possiblité et indiquons donc `1`.

```cpp
uboLayoutBinding.stageFlags = VK_SHADER_STAGE_VERTEX_BIT;
```

Nous devons aussi informer Vulkan des étapes shaders qui accèderont à cette ressource. Le champ de bits `stageFlags` permet de combiner toutes les étapes shader concernées. Vous pouvez aussi fournir la valeur `VK_SHADER_STAGE_ALL_GRAPHICS`. Nous mettons uniquement `VK_SHADER_STAGE_VERTEX_BIT`.

Tous les liens des descripteurs sont ensuite combinés en un seul objet `VkDescriptorSetLayout`. Créez pour cela un nouveau membre donnée :

```cpp
VkDescriptorSetLayout descriptorSetLayout;
VkPipelineLayout pipelineLayout;
```

Nous pouvons créer cet objet à l'aide de la fonction `vkCreateDescriptorSetLayout`. Cette fonction prend en argument une structure de type `VkDescriptorSetLayoutCreateInfo`. Elle contient un tableau contenant les structures qui décrivent les bindings :

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

Vous vous demandez peut-être pourquoi il est possible de spécifier plusieurs set de descripteurs dans cette structure, dans la mesure où un seul inclut tous les `bindings` d'une pipeline. Nous y reviendrons dans le chapitre suivant, quand nous nous intéresserons aux pools de descripteurs.

L'objet que nous avons créé ne doit être détruit que lorsque le programme se termine.

```cpp
void cleanup() {
    cleanupSwapChain();

    vkDestroyDescriptorSetLayout(device, descriptorSetLayout, nullptr);

    ...
}
```

## DescriptorPool

Les sets de descripteurs ne peuvent pas être crées directement. Il faut les allouer depuis une pool, comme les command buffers. Nous allons créer la fonction `createDescriptorPool` pour générer une pool de descripteurs.

```cpp
void initVulkan() {
    ...
    createUniformBuffer();
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

Nous allons allouer un descripteur par frame. Cette structure doit maintenant être référencée dans la structure principale `VkDescriptorPoolCreateInfo`.

```cpp
VkDescriptorPoolCreateInfo poolInfo{};
poolInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_POOL_CREATE_INFO;
poolInfo.poolSizeCount = 1;
poolInfo.pPoolSizes = &poolSize;
```

Nous devons aussi spécifier le nombre maximum de sets de descripteurs que nous sommes susceptibles d'allouer.

```cpp
poolInfo.maxSets = static_cast<uint32_t>(swapChainImages.size());
```

Ensuite on crée le nouveau membre `descriptorPool`, puis on appèle `vkCreateDescriptorPool` pour l'initialiser.

```cpp
VkDescriptorPool descriptorPool;

...

if (vkCreateDescriptorPool(device, &poolInfo, nullptr, &descriptorPool) != VK_SUCCESS) {
    throw std::runtime_error("echec de la creation de la pool de descripteurs!");
}
```

La pool doit être recréée en même temps que la swap chain. On va donc la détruire dans `cleanupSwapChain`.

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
    createUniformBuffers();
    createDescriptorPool();
    createCommandBuffers();
}
```

## DescriptorSet

Nous pouvons maintenant allouer les sets de descripteurs. Créez pour cela la fonction `createDescriptorSets` :

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

L'allocation de cette ressource passe par la création d'une structure de type `VkDescriptorSetAllocateInfo`. Vous devez indiquer la pool d'où les allouer, le nombre de sets à créer et l'organisation qu'ils doivent suivre \(`pSetLayouts`\).

```cpp
std::vector<VkDescriptorSetLayout> layouts(swapChainImages.size(), descriptorSetLayout);
VkDescriptorSetAllocateInfo allocInfo{};
allocInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_SET_ALLOCATE_INFO;
allocInfo.descriptorPool = descriptorPool;
allocInfo.descriptorSetCount = static_cast<uint32_t>(swapChainImages.size());
allocInfo.pSetLayouts = layouts.data();
```

Ajoutez un membre donnée pour garder une référence aux sets, et allouez-les avec `vkAllocateDescriptorSets` :

```cpp
VkDescriptorPool descriptorPool;
std::vector<VkDescriptorSet> descriptorSets;

...

descriptorSets.resize(swapChainImages.size());
if (vkAllocateDescriptorSets(device, &allocInfo, descriptorSets.data()) != VK_SUCCESS) {
    throw std::runtime_error("echec de l'allocation d'un set de descripteurs!");
}
```

Les descripteurs référant à un buffer doivent être configurés avec une structure de type `VkDescriptorBufferInfo`. Elle indique le buffer contenant les données, et où les données y sont stockées.

```cpp
for (size_t i = 0; i < swapChainImages.size(); i++) {
    VkDescriptorBufferInfo bufferInfo{};
    bufferInfo.buffer = uniformBuffers[i];
    bufferInfo.offset = 0;
    bufferInfo.range = sizeof(UniformBufferObject);
}
```

La configuration des descripteurs est maintenant possible, nous allons la mettre à jour a l'aide de la fonction `vkUpdateDescriptorSets`. Elle prend un tableau de `VkWriteDescriptorSet` en paramètre. Les deux premiers champs spécifient le set à mettre à jour et l'indice du binding auquel il correspond.

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

Le dernier champ que nous allons utiliser est `pBufferInfo`. Il permet de fournir `descriptorCount` structures qui configureront les descripteurs. Les autres champs correspondent aux structures qui peuvent configurer des descripteurs d'autres types. Ainsi il y aura `pImageInfo` pour les descripteurs liés aux images, et `pTexelBufferInfo` pour les descripteurs liés aux buffer views.

```cpp
descriptorWrite.pBufferInfo = &bufferInfo;
```

Enfin, nous mettons à jour notre descripteur set.

```cpp
vkUpdateDescriptorSets(device, 1, &descriptorWrite, 0, nullptr);
```

