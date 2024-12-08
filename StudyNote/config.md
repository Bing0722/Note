## 配置文件
配置文件是用来存储一些配置信息的文本文件。

配置文件的作用主要有：

1. 存储程序的设置信息
2. 存储用户的偏好设置
3. 存储程序的运行状态信息

配置文件的格式有很多种，常见的有：

1. INI文件：Windows系统中最常见的配置文件格式，使用“=”和“;”作为分隔符，可以存储键值对信息。
2. XML文件：可扩展标记语言，可以存储复杂的结构化数据。
3. JSON文件：JavaScript Object Notation，轻量级的数据交换格式，可以存储结构化数据。
4. YAML文件：YAML Ain't Markup Language，一种标记语言，可以存储结构化数据。

配置文件的命名规则一般是：

1. 名称以“.conf”结尾，表示配置文件。
2. 名称以“.ini”结尾，表示Windows系统的配置文件。
3. 名称以“.xml”结尾，表示XML格式的配置文件。
4. 名称以“.json”结尾，表示JSON格式的配置文件。
5. 名称以“.yaml”结尾，表示YAML格式的配置文件。

### 使用 JSON 格式的配置文件

JSON 格式的配置文件可以存储结构化数据，例如：

```json
{
    "username": "admin",
    "password": "123456",
    "settings": {
        "timeout": 10,
        "language": "en-US"
    }
}
```
JSON 格式的配置文件可以很方便的被解析和使用，例如：

```c++
#include <iostream>
#include <fstream>
#include <string>
#include <nlohmann/json.hpp>

int main() {
    // 打开并读取配置文件
    std::ifstream config_file("config.json");
    if(!config_file.is_open()){
        std::cerr << "Could not open config file!" << std::endl;
        return 1;
    }

    // 解析 JSON 文件
    nlohmann::json config;
    config_file >> config;
    
    // 读取配置值
    std::string username = config["username"];
    std::string password = config["password"];
    int timeout = config["setting"]["timeout"];
    std::string language = config["setting"]["language"];
    std::string name = config["username"].get<std::string>();
    
    std::cout << "Username: " << username << std::endl;
    std::cout << "Password: " << password << std::endl;
    std::cout << "Timeout: " << timeout << std::endl;
    std::cout << "Language: " << language << std::endl;  
    std::cout << "name: " << name << std::endl;

    return 0;
}
```

### 使用 YAML 格式的配置文件
YAML 格式的配置文件可以存储结构化数据，例如：

```yaml
username: admin
password: 123456
settings:
  timeout: 10
  language: en-US
```

YAML 格式的配置文件可以很方便的被解析和使用，例如：

```c++
#include <iostream>
#include <fstream>
#include <string>
#include <yaml-cpp/yaml.h>

int main() {
    // 打开并读取配置文件
    std::ifstream config_file("config.yaml");
    if(!config_file.is_open()){
        std::cerr << "Could not open config file!" << std::endl;
        return 1;
    }

    // 解析 YAML 文件
    YAML::Node config = YAML::Load(config_file);
    
    // 读取配置值
    std::string username = config["username"].as<std::string>();
    std::string password = config["password"].as<std::string>();
    int timeout = config["settings"]["timeout"].as<int>();
    std::string language = config["settings"]["language"].as<std::string>();
    std::string name = config["username"].as<std::string>();
    
    std::cout << "Username: " << username << std::endl;
    std::cout << "Password: " << password << std::endl;
    std::cout << "Timeout: " << timeout << std::endl;
    std::cout << "Language: " << language << std::endl;  
    std::cout << "name: " << name << std::endl;

    return 0;
}
```

### 使用 INI 格式的配置文件
INI 格式的配置文件可以存储键值对信息，例如：

```ini
[user]
name=admin
password=123456

[settings]
timeout=10
language=en-US
```

INI 格式的配置文件可以很方便的被解析和使用，例如：

```c++
#include <iostream>
#include <fstream>
#include <string>
#include <cstdlib>
#include <cstring>

int main() {
    // 打开并读取配置文件
    std::ifstream config_file("config.ini");
    if(!config_file.is_open()){
        std::cerr << "Could not open config file!" << std::endl;
        return 1;
    }

    // 解析 INI 文件
    char section[100];
    char key[100];
    char value[100];    
    while(config_file.getline(section, 100, '[')){
        config_file.getline(key, 100, '=');
        config_file.getline(value, 100, '\n');
        std::string section_str(section);
        std::string key_str(key);
        std::string value_str(value);
        std::cout << section_str << " " << key_str << " " << value_str << std::endl;
    }

    return 0;
}
```