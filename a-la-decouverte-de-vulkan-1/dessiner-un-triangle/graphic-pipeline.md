# Graphic Pipeline

## Introduction

 Créez la fonction `createGraphicsPipeline` et appelez-la depuis `initVulkan` après `createImageViews`. Nous travaillerons sur cette fonction dans les chapitres suivants.

```cpp
void initVulkan() {
    createInstance();
    setupDebugMessenger();
    createSurface();
    pickPhysicalDevice();
    createLogicalDevice();
    createSwapChain();
    createImageViews();
    createGraphicsPipeline();
}

...

void createGraphicsPipeline() {

}
```

### Compilation des shaders <a id="page_Compilation-des-shaders"></a>

Créez un dossier `shaders` à la racine de votre projet, puis enregistrez le vertex shader dans un fichier appelé `shader.vert` et le fragment shader dans un fichier appelé `shader.frag`. Les shaders en GLSL n'ont pas d'extension officielle mais celles-ci correspondent à l'usage communément accepté.

Le contenu de `shader.vert` devrait être:

```cpp
#version 450
#extension GL_ARB_separate_shader_objects : enable

out gl_PerVertex {
    vec4 gl_Position;
};

layout(location = 0) out vec3 fragColor;

vec2 positions[3] = vec2[](
    vec2(0.0, -0.5),
    vec2(0.5, 0.5),
    vec2(-0.5, 0.5)
);

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

 Et `shader.frag` devrait contenir :

```cpp
#version 450
#extension GL_ARB_separate_shader_objects : enable

layout(location = 0) in vec3 fragColor;

layout(location = 0) out vec4 outColor;

void main() {
    outColor = vec4(fragColor, 1.0);
}
```

Nous allons maintenant compiler ces shaders en bytecode SPIR-V à l'aide du programme `glslc`.

**Windows**

Créez un fichier `compile.bat` et copiez ceci dedans :

```cpp
C:/VulkanSDK/x.x.x.x/Bin32/glslc.exe shader.vert -o vert.spv
C:/VulkanSDK/x.x.x.x/Bin32/glslc.exe shader.frag -o frag.spv
pause
```

### Charger un shader <a id="page_Charger-un-shader"></a>

Maintenant que vous pouvez créer des shaders SPIR-V il est grand temps de les charger dans le programme et de les intégrer à la pipeline graphique. Nous allons d'abord écrire une fonction qui réalisera le chargement des données binaires à partir des fichiers.

```cpp
#include <fstream>

...

static std::vector<char> readFile(const std::string& filename) {
    std::ifstream file(filename, std::ios::ate | std::ios::binary);

    if (!file.is_open()) {
        throw std::runtime_error(std::string {"échec de l'ouverture du fichier "} + filename + "!");
    }
}
```

La fonction `readFile` lira tous les octets du fichier qu'on lui indique et les retournera dans un `vector` de caractères servant ici d'octets. L'ouverture du fichier se fait avec deux paramètres particuliers :

* `ate` : permet de commencer la lecture à la fin du fichier
* `binary` : indique que le fichier doit être lu comme des octets et que ceux-ci ne doivent pas être formatés

Commencer la lecture à la fin permet d'utiliser la position du pointeur comme indicateur de la taille totale du fichier et nous pouvons ainsi allouer un stockage suffisant :

```cpp
size_t fileSize = (size_t) file.tellg();
std::vector<char> buffer(fileSize);

file.seekg(0);
file.read(buffer.data(), fileSize);

file.close();

return buffer;
```

 Appelons maintenant cette fonction depuis `createGraphicsPipeline` pour charger les bytecodes des deux shaders :

```cpp
void createGraphicsPipeline() {
    auto vertShaderCode = readFile("shaders/vert.spv");
    auto fragShaderCode = readFile("shaders/frag.spv");
}
```

### Créer des modules shader <a id="page_Crer-des-modules-shader"></a>

Avant de passer ce code à la pipeline nous devons en faire un `VkShaderModule`. Créez pour cela une fonction `createShaderModule`.

```cpp
VkShaderModule createShaderModule(const std::vector<char>& code) {

    VkShaderModuleCreateInfo createInfo{};
    createInfo.sType = VK_STRUCTURE_TYPE_SHADER_MODULE_CREATE_INFO;
    createInfo.codeSize = code.size();
    createInfo.pCode = reinterpret_cast<const uint32_t*>(code.data());
    
    VkShaderModule shaderModule;
    if (vkCreateShaderModule(device, &createInfo, nullptr, &shaderModule) != VK_SUCCESS) {
        throw std::runtime_error("échec de la création d'un module shader!");
    }
    
    return shaderModule;
}
```

 Les modules shaders ne sont au fond qu'une fine couche autour du byte code chargé depuis les fichiers. Au moment de la création de la pipeline, les codes des shaders sont compilés et mis sur la carte. Nous pouvons donc détruire les modules dès que la pipeline est crée. Nous en ferons donc des variables locales à la fonction `createGraphicsPipeline` :

```cpp
void createGraphicsPipeline() {
    auto vertShaderModule = createShaderModule(vertShaderCode);
    fragShaderModule = createShaderModule(fragShaderCode);

    vertShaderModule = createShaderModule(vertShaderCode);
    fragShaderModule = createShaderModule(fragShaderCode);
```

 Ils doivent être libérés une fois que la pipeline est créée, juste avant que `createGraphicsPipeline` ne retourne. Ajoutez ceci à la fin de la fonction :

```cpp
    ...
    vkDestroyShaderModule(device, fragShaderModule, nullptr);
    vkDestroyShaderModule(device, vertShaderModule, nullptr);
}
```

### Création des étapes shader <a id="page_Cration-des-tapes-shader"></a>

Nous devons assigner une étape shader aux modules que nous avons crées. Nous allons utiliser une structure du type `VkPipelineShaderStageCreateInfo` pour cela.

Nous allons d'abord remplir cette structure pour le vertex shader, une fois de plus dans la fonction `createGraphicsPipeline`.

```cpp
VkPipelineShaderStageCreateInfo vertShaderStageInfo{};
vertShaderStageInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_SHADER_STAGE_CREATE_INFO;
vertShaderStageInfo.stage = VK_SHADER_STAGE_VERTEX_BIT;

vertShaderStageInfo.module = vertShaderModule;
vertShaderStageInfo.pName = "main";
```

Modifier la structure pour qu'elle corresponde au fragment shader est très simple :

```cpp
VkPipelineShaderStageCreateInfo fragShaderStageInfo{};
fragShaderStageInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_SHADER_STAGE_CREATE_INFO;
fragShaderStageInfo.stage = VK_SHADER_STAGE_FRAGMENT_BIT;
fragShaderStageInfo.module = fragShaderModule;
fragShaderStageInfo.pName = "main";
```

Intégrez ces deux valeurs dans un tableau que nous utiliserons plus tard et vous aurez fini ce chapitre!

```cpp
VkPipelineShaderStageCreateInfo shaderStages[] = {vertShaderStageInfo, fragShaderStageInfo};
```

C'est tout ce que nous dirons sur les étapes programmables de la pipeline. Dans le prochain chapitre nous verrons les étapes à fonction fixée.

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-10-modules-shaders.cpp" %}

{% file src="../../.gitbook/assets/shader.vert" %}

{% file src="../../.gitbook/assets/shader.frag" %}

