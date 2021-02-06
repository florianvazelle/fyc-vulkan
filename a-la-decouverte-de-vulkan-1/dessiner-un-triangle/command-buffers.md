# Command Buffers

Les queues sont les composants qui vont traiter les différentes tâches de l'application. Ces tâches vont être regroupées par paquets appelés **Command Buffers**. Une fois que l'application à crée ces command buffers, elle les envois aux différentes queues pour être exécuté.

Les Command Buffers ne sont pas crée directement, ils doivent être alloué par des Command Pools. Pour créer une Command Pool, on utilise la fonction `vkCreateCommandPool()`. 

```cpp
VkResult vkCreateCommandPool(
    VkDevice                                    device,
    const VkCommandPoolCreateInfo*              pCreateInfo,
    const VkAllocationCallbacks*                pAllocator,
    VkCommandPool*                              pCommandPool);
```

On renseigne le device qui possédera la nouvelle pool ainsi qu'un pointeur vers  les informations de création de la pool.

```cpp
typedef struct VkCommandPoolCreateInfo {
    VkStructureType             sType;
    const void*                 pNext;
    VkCommandPoolCreateFlags    flags;
    uint32_t                    queueFamilyIndex;
} VkCommandPoolCreateInfo;
```

 Les flags de création vont définir si les command buffers seront persistants ou non afin d'allouer la mémoire correctement. Avec certains paramètres il est également possible de recycler les command buffers après leur utilisation.

Préciser le bon type d'allocation mémoire permet d'éviter la fragmentation car les command buffers seront alloué régulièrement.

A la fin du programme il faudra également désallouer la mémoire allouée pour ces command pools avec la fonction `vkDestroyCommandPool()`.

```cpp
void vkDestroyCommandPool(
    VkDevice                                    device,
    VkCommandPool                               commandPool,
    const VkAllocationCallbacks*                pAllocator);
```

Une fois que la command Pool est crée, on peut finalement allouer des command Buffers qui servirons à empiler nos traitements. Pour cela on utilise la fonction `vkAllocateCommandBuffers()`.

```cpp
VkResult vkAllocateCommandBuffers(
    VkDevice                                    device,
    const VkCommandBufferAllocateInfo*          pAllocateInfo,
    VkCommandBuffer*                            pCommandBuffers);
```

La fonction prend en paramètre le device ainsi que les informations d'allocation du command Buffer. On a plus qu'à réutiliser le pointeur `VkCommandBuffer` passé en paramètre.

Pour spécifier les paramètres d'allocation, on utilise la structure `VkCommandBufferAllocateInfo`.

```cpp
typedef struct VkCommandBufferAllocateInfo {
    VkStructureType         sType;
    const void*             pNext;
    VkCommandPool           commandPool;
    VkCommandBufferLevel    level;
    uint32_t                commandBufferCount;
} VkCommandBufferAllocateInfo;
```

Ici les paramètres sont assez explicites, on passe la command Pool sur laquelle vont etre alloué les command Buffers, le niveau de command Buffer \(Primary ou Secondary\) en effet, un command Buffer "Primary" peut appeler un "Secondary" lors de son exécution. Le dernier paramètre étant le nombre de command Buffers à allouer depuis la pool.

Après utilisation, les command buffers doivent être désalloués de la mémoire. On utilise la fonction `vkFreeCommandBuffers()`.

```cpp
void vkFreeCommandBuffers(
    VkDevice                                    device,
    VkCommandPool                               commandPool,
    uint32_t                                    commandBufferCount,
    const VkCommandBuffer*                      pCommandBuffers);
```

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-15-command-buffers.cpp" %}

{% embed url="https://youtu.be/XhOox6h1z9s" %}

