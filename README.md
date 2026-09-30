# imgui-test

Projeto de teste com **Dear ImGui + GLFW + OpenGL 3 + GLAD**, compilado com CMake.

## Estrutura do projeto

```
imgui-test/
├── CMakeLists.txt
├── README.md
├── .gitignore
├── external/
│   ├── imgui/              # código do Dear ImGui (todos os arquivos)
│   └── glad/
│       ├── include/
│       │   ├── glad/glad.h
│       │   └── KHR/khrplatform.h
│       └── src/glad.c
└── src/
    └── main.cpp
```

**Regra geral:** código de terceiros (ImGui, GLAD) fica em `external/`;
o código do projeto fica em `src/`.

## Dependências

Fedora:

```bash
sudo dnf install cmake gcc gcc-c++ make glfw-devel mesa-libGL-devel
```

(Em Debian/Ubuntu: `sudo apt install cmake build-essential libglfw3-dev`.)

## Compilar e executar

```bash
cmake -S . -B build
cmake --build build -j
./build/imgui-test
```

Para recompilar do zero:

```bash
rm -rf build
```

## CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.16)
project(imgui-test LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Debug)
endif()

# --- Dependências do sistema ---
find_package(OpenGL REQUIRED)
find_package(glfw3 REQUIRED)

# --- GLAD ---
add_library(glad STATIC external/glad/src/glad.c)
target_include_directories(glad PUBLIC external/glad/include)

# --- Dear ImGui ---
set(IMGUI_DIR ${CMAKE_CURRENT_SOURCE_DIR}/external/imgui)

add_library(imgui STATIC
    ${IMGUI_DIR}/imgui.cpp
    ${IMGUI_DIR}/imgui_demo.cpp
    ${IMGUI_DIR}/imgui_draw.cpp
    ${IMGUI_DIR}/imgui_tables.cpp
    ${IMGUI_DIR}/imgui_widgets.cpp
    ${IMGUI_DIR}/imgui_impl_glfw.cpp
    ${IMGUI_DIR}/imgui_impl_opengl3.cpp
)
target_include_directories(imgui PUBLIC ${IMGUI_DIR})
target_link_libraries(imgui PUBLIC glfw OpenGL::GL ${CMAKE_DL_LIBS})

# --- Executável ---
add_executable(${PROJECT_NAME} src/main.cpp)
target_link_libraries(${PROJECT_NAME} PRIVATE imgui glad glfw OpenGL::GL)

if(MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE /W4)
else()
    target_compile_options(${PROJECT_NAME} PRIVATE -Wall -Wextra)
endif()
```

Só os arquivos `.cpp` entram no CMake; os `.h` não precisam ser listados.

## Includes no main.cpp

Como o CMake já expõe `external/imgui` e `external/glad/include` como
diretórios de include, **não use prefixo de pasta**:

```cpp
#include <glad/glad.h>      // GLAD sempre ANTES do GLFW
#include <GLFW/glfw3.h>

#include "imgui.h"
#include "imgui_impl_glfw.h"
#include "imgui_impl_opengl3.h"
```

Errado (causa `fatal error: imgui/imgui.h: No such file or directory`):

```cpp
#include "imgui/imgui.h"    // ❌
```

Alternativa, se preferir manter `imgui/imgui.h` no código: troque o include
directory do ImGui no CMake por `${CMAKE_CURRENT_SOURCE_DIR}/external`.

## Problemas comuns

| Problema | Causa / solução |
|---|---|
| `imgui/imgui.h: No such file or directory` | Include com prefixo `imgui/`. Use `#include "imgui.h"`. |
| Erros de redefinição do OpenGL | Inclua `glad/glad.h` antes de `GLFW/glfw3.h`. |
| `Could not find glfw3` | No Fedora: `sudo dnf install glfw-devel` (no Ubuntu: `libglfw3-dev`). |
| CMake com cache antigo/estranho | `rm -rf build` e rode o `cmake -S . -B build` de novo. |

## .gitignore sugerido

```
build/
.cache/
compile_commands.json
```
