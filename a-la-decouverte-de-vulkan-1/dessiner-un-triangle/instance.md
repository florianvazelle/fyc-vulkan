# Instance

Tout d'abord, nous devons créer une instance de Vulkan. Un `VkInstance` est un objet qui contient toutes les informations dont l'application Vulkan a besoin pour fonctionner. Contrairement à OpenGL, Vulkan n'a pas d'état global. Pour cette raison, nous devons à la place stocker nos états dans cet objet. Dans ce chapitre, nous allons commencer un cours que nous utiliserons pour le reste du livre.

```cpp
#ifndef INSTANCE_HPP
#define INSTANCE_HPP

namespace vks {

  class Instance {
  public:
    Instance(const char* appName, const char* engineName);
    Instance() = delete;
    ~Instance();

    inline const VkInstance& handle() const { return m_instance; }

  private:
    VkInstance m_instance;
  };

}  // namespace vks

#endif  // INSTANCE_HPP
```

```cpp
#include <vks/Instance.hpp>

using namespace vks;

Instance::Instance(const char* appName, const char* engineName)
    : m_instance(VK_NULL_HANDLE) {

  VkApplicationInfo appInfo{};
  appInfo.sType = VK_STRUCTURE_TYPE_APPLICATION_INFO;
  appInfo.pApplicationName = appName;
  appInfo.applicationVersion = VK_MAKE_VERSION(1, 0, 0);
  appInfo.pEngineName = engineName;
  appInfo.engineVersion = VK_MAKE_VERSION(1, 0, 0);
  appInfo.apiVersion = VK_API_VERSION_1_0;

  VkInstanceCreateInfo createInfo{};
  createInfo.sType = VK_STRUCTURE_TYPE_INSTANCE_CREATE_INFO;
  createInfo.pApplicationInfo = &appInfo;

  createInfo.enabledLayerCount = 0;
  createInfo.pNext = nullptr;

  if (vkCreateInstance(&createInfo, nullptr, &m_instance) != VK_SUCCESS) {
    throw std::runtime_error("failed to create instance!");
  }
}

Instance::~Instance() { vkDestroyInstance(m_instance, nullptr); }
```

**Vidéo / Code :**

{% file src="../../.gitbook/assets/part-3-instance.cpp" %}

