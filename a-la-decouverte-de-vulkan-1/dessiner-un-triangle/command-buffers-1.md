# Command Buffers

### Command Pool <a id="page_Command-pools"></a>

Nous devons créer une _command pool_ avant de pouvoir créer les command buffers. Les command pools gèrent la mémoire utilisée par les buffers, et c'est de fait les command pools qui nous instancient les command buffers. Ajoutez un nouveau membre donnée à la classe de type [`VkCommandPool`](https://www.khronos.org/registry/vulkan/specs/1.0/man/html/VkCommandPool.html) :

```cpp
VkCommandPool commandPool;
```

Créez ensuite la fonction `createCommandPool` et appelez-la depuis `initVulkan` après la création du framebuffer.

```cpp
void initVulkan() {
    createInstance();
    setupDebugMessenger();
    createSurface();
    pickPhysicalDevice();
    createLogicalDevice();
    createSwapChain();
    createImageViews();
    createRenderPass();
    createGraphicsPipeline();
    createFramebuffers();
    createCommandPool();
}

...

void createCommandPool() {

}
```

La création d'une command pool ne nécessite que deux paramètres :

```cpp
QueueFamilyIndices queueFamilyIndices = findQueueFamilies(physicalDevice);

VkCommandPoolCreateInfo poolInfo{};
poolInfo.sType = VK_STRUCTURE_TYPE_COMMAND_POOL_CREATE_INFO;
poolInfo.queueFamilyIndex = queueFamilyIndices.graphicsFamily.value();
poolInfo.flags = 0; // Optionel
```

Nous n'enregistrerons les command buffers qu'une seule fois au début du programme, nous n'aurons donc pas besoin de ces fonctionnalités.

```cpp
if (vkCreateCommandPool(device, &poolInfo, nullptr, &commandPool) != VK_SUCCESS) {
    throw std::runtime_error("échec de la création d'une command pool!");
}
```

 Terminez la création de la command pool à l'aide de la fonction [`vkCreateComandPool`](https://www.khronos.org/registry/vulkan/specs/1.0/man/html/vkCreateComandPool.html). Elle ne comprend pas de paramètre particulier. Les commandes seront utilisées tout au long du programme pour tout affichage, nous ne devons donc la détruire que dans la fonction `cleanup` :

```cpp
void cleanup() {
    vkDestroyCommandPool(device, commandPool, nullptr);

    ...
}
```

### Allocation des Command Buffers <a id="page_Allocation-des-command-buffers"></a>

Nous pouvons maintenant allouer des command buffers et enregistrer les commandes d'affichage. Dans la mesure où l'une des commandes consiste à lier un framebuffer nous devrons les enregistrer pour chacune des images de la swap chain. Créez pour cela une liste de [`VkCommandBuffer`](https://www.khronos.org/registry/vulkan/specs/1.0/man/html/VkCommandBuffer.html) et stockez-la dans un membre donnée de la classe. Les command buffers sont libérés avec la destruction de leur command pool, nous n'avons donc pas à faire ce travail.

```cpp
std::vector<VkCommandBuffer> commandBuffers;
```

Commençons maintenant à travailler sur notre fonction `createCommandBuffers` qui allouera et enregistrera les command buffers pour chacune des images de la swap chain.

```cpp
void initVulkan() {
    createInstance();
    setupDebugMessenger();
    createSurface();
    pickPhysicalDevice();
    createLogicalDevice();
    createSwapChain();
    createImageViews();
    createRenderPass();
    createGraphicsPipeline();
    createFramebuffers();
    createCommandPool();
    createCommandBuffers();
}

...

void createCommandBuffers() {
    commandBuffers.resize(swapChainFramebuffers.size());
}
```

Les command buffers sont alloués par la fonction [`vkAllocateCommandBuffers`](https://www.khronos.org/registry/vulkan/specs/1.0/man/html/vkAllocateCommandBuffers.html) qui prend en paramètre une structure du type [`VkCommandBufferAllocateInfo`](https://www.khronos.org/registry/vulkan/specs/1.0/man/html/VkCommandBufferAllocateInfo.html). Cette structure spécifie la command pool et le nombre de buffers à allouer depuis celle-ci:

```cpp
VkCommandBufferAllocateInfo allocInfo{};
allocInfo.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_ALLOCATE_INFO;
allocInfo.commandPool = commandPool;
allocInfo.level = VK_COMMAND_BUFFER_LEVEL_PRIMARY;
allocInfo.commandBufferCount = (uint32_t) commandBuffers.size();

if (vkAllocateCommandBuffers(device, &allocInfo, commandBuffers.data()) != VK_SUCCESS) {
    throw std::runtime_error("échec de l'allocation de command buffers!");
}
```

### Enregistrement des Commandes <a id="page_Dbut-de-l-enregistrement-des-commandes"></a>

Nous commençons l'enregistrement des commandes en appelant [`vkBeginCommandBuffer`](https://www.khronos.org/registry/vulkan/specs/1.0/man/html/vkBeginCommandBuffer.html). Cette fonction prend une petite structure du type [`VkCommandBufferBeginInfo`](https://www.khronos.org/registry/vulkan/specs/1.0/man/html/VkCommandBufferBeginInfo.html) en argument, permettant d'indiquer quelques détails sur l'utilisation du command buffer.

```cpp
for (size_t i = 0; i < commandBuffers.size(); i++) {
    VkCommandBufferBeginInfo beginInfo{};
    beginInfo.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_BEGIN_INFO;
    beginInfo.flags = 0; // Optionnel
    beginInfo.pInheritanceInfo = nullptr; // Optionel

    if (vkBeginCommandBuffer(commandBuffers[i], &beginInfo) != VK_SUCCESS) {
        throw std::runtime_error("erreur au début de l'enregistrement d'un command buffer!");
    }
}
```

### Commencer une render pass <a id="page_Commencer-une-render-pass"></a>

L'affichage commence par le lancement de la render pass réalisé par [`vkCmdBeginRenderPass`](https://www.khronos.org/registry/vulkan/specs/1.0/man/html/vkCmdBeginRenderPass.html). La passe est configurée à l'aide des paramètres remplis dans une structure de type [`VkRenderPassBeginInfo`](https://www.khronos.org/registry/vulkan/specs/1.0/man/html/VkRenderPassBeginInfo.html).

```cpp
VkRenderPassBeginInfo renderPassInfo{};
renderPassInfo.sType = VK_STRUCTURE_TYPE_RENDER_PASS_BEGIN_INFO;
renderPassInfo.renderPass = renderPass;
renderPassInfo.framebuffer = swapChainFramebuffers[i];

renderPassInfo.renderArea.offset = {0, 0};
renderPassInfo.renderArea.extent = swapChainExtent;

VkClearValue clearColor = {0.0f, 0.0f, 0.0f, 1.0f};
renderPassInfo.clearValueCount = 1;
renderPassInfo.pClearValues = &clearColor;

vkCmdBeginRenderPass(commandBuffers[i], &renderPassInfo, VK_SUBPASS_CONTENTS_INLINE);
```

### Commandes d'affichage basiques <a id="page_Commandes-d-affichage-basiques"></a>

Nous pouvons maintenant activer la pipeline graphique :

```cpp
vkCmdBindPipeline(commandBuffers[i], VK_PIPELINE_BIND_POINT_GRAPHICS, graphicsPipeline);

vkCmdDraw(commandBuffers[i], 3, 1, 0, 0);
```

### Finitions <a id="page_Finitions"></a>

La render pass peut ensuite être terminée :

```cpp
vkCmdEndRenderPass(commandBuffers[i]);

if (vkEndCommandBuffer(commandBuffers[i]) != VK_SUCCESS) {
    throw std::runtime_error("échec de l'enregistrement d'un command buffer!");
}
```

Dans le prochain chapitre nous écrirons le code pour la boucle principale. Elle récupérera une image de la swap chain, exécutera le bon command buffer et retournera l'image complète à la swap chain.

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-15-command-buffers.cpp" %}

{% embed url="https://youtu.be/XhOox6h1z9s" %}

