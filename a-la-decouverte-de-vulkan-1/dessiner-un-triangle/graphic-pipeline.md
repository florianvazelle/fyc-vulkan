# Graphic Pipeline

Le Graphic Pipeline, au sens général, définit toute la tuyauterie, toutes les opérations qu’exécutent notre GPU pour nous sortir un rendu. Pour résumé, dans cette tuyauterie :

* on dépose en entrée des Vertex/Index Buffers
* on obtient en sortie des pixels rendus sur un Framebuffer

Le Graphic Pipeline, au sens de Vulkan, est très complexe et nous n'allons donc en voir qu'une version simplifiée.

![](../../.gitbook/assets/bitmap.png)

## Shader

Nous aurons donc deux étapes dites de shader. Créons un dossier `shaders` contenant les deux shaders, qui sont très classiques, ci-dessous :

{% code title="shader.vert" %}
```cpp
#version 450
#extension GL_ARB_separate_shader_objects : enable

out gl_PerVertex {
    vec4 gl_Position;
};

layout(location = 0) out vec3 fragColor;

// On stock en dure les coordonnées du triangle
vec2 positions[3] = vec2[](
    vec2(0.0, -0.5),
    vec2(0.5, 0.5),
    vec2(-0.5, 0.5)
);

// Et ces couleurs
vec3 colors[3] = vec3[](
    vec3(1.0, 0.0, 0.0),
    vec3(0.0, 1.0, 0.0),
    vec3(0.0, 0.0, 1.0)
);

void main() {
    gl_Position = vec4(positions[gl_VertexIndex], 0.0, 1.0);
    fragColor = colors[gl_VertexIndex];
}
```
{% endcode %}

{% code title="shader.frag" %}
```cpp
#version 450
#extension GL_ARB_separate_shader_objects : enable

layout(location = 0) in vec3 fragColor;

layout(location = 0) out vec4 outColor;

void main() {
    outColor = vec4(fragColor, 1.0);
}
```
{% endcode %}

### Compilation en SPIR-V

Nous en avions déjà parlé, en Vulkan il faut compiler les shaders en bytecode SPIR-V. Le SPIR-V a un énorme intérêt d'interopérabilité des shaders. Il y a plusieurs outils pour cela, voici quelques exemples :

* pour la compilation GLSL : `glslc`, `glslangValidator`
* pour la compilation des compute shader OpenCL : `clspv`

![The SPIR-V Open Source Ecosystem](../../.gitbook/assets/image.png)

## Pipeline

Créons une fonction `createGraphicsPipeline` et appelons-la depuis `initVulkan` après `createImageViews`. 

```cpp
void initVulkan() {
    ...
    createImageViews();
    createGraphicsPipeline();
}
```

### Pipeline Layout

Un Pipeline Layout contient les informations sur les entrées de shader \(DescritpeurSet\). C'est ici aussi que nous pouvons configurer des Push Constants. Mais pour le moment, nous n'en avons pas besoin.

```cpp
VkPipelineLayout pipelineLayout;

const VkPipelineLayoutCreateInfo pipelineLayoutInfo = {
    .sType          = VK_STRUCTURE_TYPE_PIPELINE_LAYOUT_CREATE_INFO,
    .setLayoutCount = 0,
    .pushConstantRangeCount = 0,
};

if (vkCreatePipelineLayout(device, &pipelineLayoutInfo, nullptr, &pipelineLayout) != VK_SUCCESS) {
    throw std::runtime_error("Pipeline Layout creation failed");
}
```

### Graphic Pipeline

Nous allons maintenant définir notre Graphic Pipeline. Il s'agit d'un objet avec de nombreuses structures de configurations différentes.

#### Vertex Input

`VkPipelineVertexInputStateCreateInfo` contient les informations sur les Vertex Buffers. 

{% hint style="info" %}
C'est l'équivalent des Vertex Array Object sous OpenGL
{% endhint %}

Cependant dans ce tutoriel, nous écrivons nos données en dur dans le shader, donc nous l'initialiserons avec un état vide.

```cpp
VkPipelineVertexInputStateCreateInfo vertexInputInfo = {
    .sType = VK_STRUCTURE_TYPE_PIPELINE_VERTEX_INPUT_STATE_CREATE_INFO,
    .vertexBindingDescriptionCount   = 0,
    .vertexAttributeDescriptionCount = 0,
};
```

#### Input Assembly

`VkPipelineInputAssemblyStateCreateInfo` contient la configuration du type de topologie qui sera dessinée, c'est-à-dire qu'on spécifie comment le GPU doit assembler les données en entrée.

```cpp
VkPipelineInputAssemblyStateCreateInfo inputAssembly = {
    .sType                  = VK_STRUCTURE_TYPE_PIPELINE_INPUT_ASSEMBLY_STATE_CREATE_INFO,
    .topology               = VK_PRIMITIVE_TOPOLOGY_TRIANGLE_LIST,
    .primitiveRestartEnable = VK_FALSE,
};
```

#### Viewport

```cpp
// Pipeline viewport
VkViewport viewport = {
    .x      = 0.0f,
    .y      = 0.0f,
    .width  = static_cast<float>(swapChainExtent.width),
    .height = static_cast<float>(swapChainExtent.height),
    // Depth buffer range
    .minDepth = 0.0f,
    .maxDepth = 1.0f,
};

// Pixel boundary cutoff
VkRect2D scissor = {
    .offset  = {0, 0},
    .extent = swapChainExtent,
};

// Combine viewport(s) and scissor(s) (some graphics cards allow multiple of each)
VkPipelineViewportStateCreateInfo viewportState = {
    .sType         = VK_STRUCTURE_TYPE_PIPELINE_VIEWPORT_STATE_CREATE_INFO,
    .viewportCount = 1,
    .pViewports    = &viewport,
    .scissorCount  = 1,
    .pScissors     = &scissor,
};
```

#### Rasterizer

`VkPipelineRasterizationStateCreateInfo` contient la configuration de notre rastérisation. Par exemple, on va pouvoir activer ou désactiver la sélection de la face arrière, définir la largeur des lignes ou le wireframe ...

```cpp
VkPipelineRasterizationStateCreateInfo rasterizer = {
    .sType = VK_STRUCTURE_TYPE_PIPELINE_RASTERIZATION_STATE_CREATE_INFO,
    // Clip fragments instead of clipping them to near and far planes
    .depthClampEnable = VK_FALSE,
    // Don't allow the rasterizer to discard geometry
    .rasterizerDiscardEnable = VK_FALSE,
    // Fill fragments
    .polygonMode = VK_POLYGON_MODE_FILL,
    .cullMode    = VK_CULL_MODE_BACK_BIT,
    .frontFace   = VK_FRONT_FACE_COUNTER_CLOCKWISE,
    // Bias depth values
    // This is good for shadow mapping, but we're not doing that currently
    // so we'll disable for now
    .depthBiasEnable = VK_FALSE,
    .lineWidth       = 1.0f,
};
```

#### Multisampling

La structure `VkPipelineMultisampleCreateInfo` permet de configurer le multisampling qui permet de réaliser de l'anti-aliasing.

```cpp
VkPipelineMultisampleStateCreateInfo multisampling = {
    .sType                = VK_STRUCTURE_TYPE_PIPELINE_MULTISAMPLE_STATE_CREATE_INFO,
    .rasterizationSamples = VK_SAMPLE_COUNT_1_BIT,
    .sampleShadingEnable  = VK_FALSE,
    .minSampleShading     = 1.0f,
};
```

#### Color Blending

`VkPipelineColorBlendAttachmentState` contrôle la manière dont ce pipeline combine la couleur donnée par notre fragment shader et la couleur déjà présente dans le framebuffer.

```cpp
VkPipelineColorBlendAttachmentState colorBlendAttachment = {
    // Disable blending
    .blendEnable    = VK_FALSE,
    .colorWriteMask = VK_COLOR_COMPONENT_R_BIT | VK_COLOR_COMPONENT_G_BIT | VK_COLOR_COMPONENT_B_BIT | VK_COLOR_COMPONENT_A_BIT,
};

VkPipelineColorBlendStateCreateInfo colorBlending = {
    .sType           = VK_STRUCTURE_TYPE_PIPELINE_COLOR_BLEND_STATE_CREATE_INFO,
    .attachmentCount = 1,
    .pAttachments    = &colorBlendAttachment,
};
```

#### Charger les shaders

Nos shaders compilés en SPIR-V, il va falloir les lire :

```cpp
static std::vector<char> readFile(const std::string& filename) {
    std::ifstream file(filename, std::ios::ate | std::ios::binary);
    
    if (!file.is_open()) {
        throw std::runtime_error("failed to open file!");
    }
    
    size_t fileSize = (size_t) file.tellg();
    std::vector<char> buffer(fileSize);
    
    file.seekg(0);
    file.read(buffer.data(), fileSize);
    
    file.close();
    
    return buffer;
}
```

Et les transformer en `VkShaderModule` :

```cpp
VkShaderModule createShaderModule(const std::vector<char>& code) {
    VkShaderModuleCreateInfo createInfo{};
    createInfo.sType = VK_STRUCTURE_TYPE_SHADER_MODULE_CREATE_INFO;
    createInfo.codeSize = code.size();
    createInfo.pCode = reinterpret_cast<const uint32_t*>(code.data());
    
    VkShaderModule shaderModule;
    if (vkCreateShaderModule(device, &createInfo, nullptr, &shaderModule) != VK_SUCCESS) {
        throw std::runtime_error("failed to create shader module!");
    }
    
    return shaderModule;
}
```

Pour finir, initialisons un `std::vector<VkPipelineShaderStageCreateInfo> shaderStages` :

```cpp
auto vertShaderCode = readFile("shaders/vert.spv");
auto fragShaderCode = readFile("shaders/frag.spv");

VkShaderModule vertShaderModule = createShaderModule(vertShaderCode);
VkShaderModule fragShaderModule = createShaderModule(fragShaderCode);

VkPipelineShaderStageCreateInfo vertShaderStageInfo = {
    .sType = VK_STRUCTURE_TYPE_PIPELINE_SHADER_STAGE_CREATE_INFO;
    .stage = VK_SHADER_STAGE_VERTEX_BIT;
    .module = vertShaderModule;
    .pName = "main";
};

VkPipelineShaderStageCreateInfo fragShaderStageInfo = {
    .sType = VK_STRUCTURE_TYPE_PIPELINE_SHADER_STAGE_CREATE_INFO,
    .stage = VK_SHADER_STAGE_FRAGMENT_BIT,
    .module = fragShaderModule,
    .pName = "main",
};

std::vector<VkPipelineShaderStageCreateInfo> shaderStages = {vertShaderStageInfo, fragShaderStageInfo};
```

#### Création

Ensuite, on combine l'ensemble de nos objets dans une structure `VkGraphicsPipelineCreateInfo`.

```cpp
VkPipeline graphicsPipeline;

const VkGraphicsPipelineCreateInfo pipelineInfo = {
    .sType               = VK_STRUCTURE_TYPE_GRAPHICS_PIPELINE_CREATE_INFO,
    .stageCount          = 2,
    .pStages             = shaderStages.data(),
    .pVertexInputState   = &vertexInputInfo,
    .pInputAssemblyState = &inputAssembly,
    .pViewportState      = &viewportState,
    .pRasterizationState = &rasterizer,
    .pMultisampleState   = &multisampling,
    .pColorBlendState    = &colorBlending,
    .layout              = m_layout,
    .renderPass          = m_renderPass.handle(),
    .subpass             = 0, // Pipeline will be used in first sub pass
    .basePipelineHandle  = VK_NULL_HANDLE,
    .basePipelineIndex   = -1,
};

if (vkCreateGraphicsPipelines(device, VK_NULL_HANDLE, 1, &pipelineInfo, nullptr, &graphicsPipeline) != VK_SUCCESS) {
  throw std::runtime_error("Graphics Pipeline creation failed");
}
```

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-10-modules-shaders.cpp" %}

{% file src="../../.gitbook/assets/shader.vert" %}

{% file src="../../.gitbook/assets/shader.frag" %}

{% embed url="https://youtu.be/sMruz9-IjbE" %}

{% file src="../../.gitbook/assets/part-11-fonctions-fixees \(1\).cpp" %}

{% embed url="https://youtu.be/4GNpdLdxFr4" %}

