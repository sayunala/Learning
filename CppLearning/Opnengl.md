# OpenGL

 https://docs.gl/

## 1 使用GLEW与GLFW

### 1.1 传统OpenGL创建三角形

```cpp

        glBegin(GL_TRIANGLES);
        glVertex2f(0.5f, 0.5f);
        glVertex2f(0, 0);
        glVertex2f(1.0f, 0);
        glEnd();
```

+ 使用glew.h必须在gl.h之前
+ 同时glewInit必须在 现有gl上下文之后 本例中如 glfwMakeContextCurrent(window);

```cpp
#include<GL/glew.h>
#include <GLFW/glfw3.h>
#include<iostream>
int main(void)
{
    /* Initialize the library */

    GLFWwindow* window;
    if (!glfwInit())
        return -1;
    /* Create a windowed m ode window and its OpenGL context */
    window = glfwCreateWindow(640, 480, "Hello World", NULL, NULL);
    if (!window)
    {
        glfwTerminate();
        return -1;
    }

    /* Make the window's context current */
    glfwMakeContextCurrent(window);

    if (glewInit() != GLEW_OK)
        std::cout << "Error!" << std::endl;
 
    unsigned int a;
    glGenBuffers(1, &a);
    /* Loop until the user closes the window */
    while (!glfwWindowShouldClose(window))
    {
        /* Render here */
        glClear(GL_COLOR_BUFFER_BIT);

        glBegin(GL_TRIANGLES);
        glVertex2f(0.5f, 0.5f);
        glVertex2f(0, 0);
        glVertex2f(1.0f, 0);
        glEnd();
        /* Swap front and back buffers */
        glfwSwapBuffers(window);

        /* Poll for and process events */
        glfwPollEvents();
    }
    glfwTerminate();
    return 0;
}
```

### 1.2 现代OpenGL创建三角形

+ 顶点缓冲区 buffer，现存中存bytes的地方

+ 顶点着色器，运行在显卡上的程序，编写的可以在显卡上运行的代码

  **OpenGL具体的操作就是一个状态机，我们所做的就是设置一系列的状态。**

  每一个东西都有一个**编号**比如给我们创建的buffer绑定一个编号，之后的操作都是使用编号来进行操作。

  #### 1.2.1 创建顶点缓冲区

|       API        |                    参数                    |         功能         |
| :--------------: | :----------------------------------------: | :------------------: |
|   glGenBuffers   |    （bufferCount,unsigned int *buffer)     |      产生buffer      |
| **glBindBuffer** |             (缓存类型,buffer);             |    绑定当前buffer    |
|   glBufferData   | (缓存类型，缓存大小，数据首地址，缓存模式) |   将数据存入buffer   |
|   glDrawArrays   |                                            | 无索数据时的DrawCall |
|  glDrawElements  |                                            | 有索引数据的DrawCall |

```cpp
unsigned int buffer;
glGenBuffers(1, &buffer);//生成bufer
float position[6] = {
    0.5f, 0.5,
    0.0f, 0.0f,
    1.0f, 0.0f
};
glBindBuffer(GL_ARRAY_BUFFER,buffer);//绑定当前buffer
glBufferData(GL_ARRAY_BUFFER, 6 * sizeof(float), position, GL_STATIC_DRAW);//为buffer存入数据
```

#### 1.2.2 顶点属性和布局

| glVertexAttribPointer | index          | size        | type                   | normalized | stride             | pointer                                  |
| :-------------------- | -------------- | ----------- | ---------------------- | ---------- | ------------------ | ---------------------------------------- |
| 设置顶点属性和布局    | 每个属性的编号 | 属性个数0-4 | 属性类型int,float .etc | 是否归一化 | 每个顶点所占字节数 | 一个顶点内当前属性的开始地址，字节为单位 |

```cpp
glEnableVertexAttribArray(0);
glVertexAttribPointer(0, 2, GL_FLOAT, GL_FALSE,sizeof(float) * 2, 0);
```

#### 1.2.3 着色器

着色器（shader）本质就是运行在GPU上的一段程序。

+ **片段着色器**（fragment shaders or pixel shaders）
+ **顶点着色器**：目的就是确定顶点位置的一段程序。对于多少个顶点调用多少次顶点着色器
+ 细分曲面着色器（tessellation shaders）
+ 几何着色器



|        API         |                   参数                   |    返回    |                功能                |
| :----------------: | :--------------------------------------: | :--------: | :--------------------------------: |
|   glCreateProgrm   |                    无                    | program id |        创建运行着色器的程序        |
|   glCreateShader   |                着色器类型                | shader id  |             创建着色器             |
|   glShaderSource   |      shader_id,shader_count,source       |            |           绑定着色器代码           |
|  glCompileShader   |                shader_id                 |            |           编译着色器代码           |
|   glGetShaderiv    |     shader_id, shadre_Lenum, result      |            | 根据shader_Lenum 获取想要的result  |
| glGetShaderInfoLog | shader_id, Info_lengt, Info_length, Info |            |                                    |
|   glDeleteShader   |                shader_id                 |            |             删除着色器             |
|                    |                                          |            |                                    |
|  glAttachShader()  |          program_id, shader_id           |            |    绑定program_id, 和shader_id     |
|  glLinkProgram()   |                program_id                |            | GPU中连接到program_id指定的program |
| glValidateProgram  |                program_id                |            |       program_id的有效性验证       |
|    glUseProgram    |                program_id                |            |        在GPU上运行相应程序         |

```cpp
static std::tuple<std::string, std::string>ParseShader(const std::string& filepath) {
    //解析着色器
    std::ifstream stream(filepath);
    std::string line;
    std::stringstream ss[2];
    enum class ShaderType {
        None = -1,
        Vertex = 0,
        Fragment = 1
    };
    ShaderType type = ShaderType::None;
    while (getline(stream,line))
    {
        if (line.find("#shader") != std::string::npos) {
            if (line.find("vertex") != std::string::npos)
                type = ShaderType::Vertex;
            else
                type = ShaderType::Fragment;
        }
        else {
            ss[(int)type] << line << "\n";//读入line
        }
    }
    return std::tuple<std::string, std::string>(ss[0].str(), ss[1].str());
    
}
static unsigned int CompileShader(unsigned int type, const std::string& source) {
    unsigned int id = glCreateShader(type);
    const char* src = source.c_str();
    glShaderSource(id, 1, &src, nullptr);//绑定着色器代码，生成着色器
    glCompileShader(id);//编译id表示的着色器

    int result;
    glGetShaderiv(id, GL_COMPILE_STATUS, &result);//获取当前编译状态
    if (result == GL_FALSE) {
        int length;
        glGetShaderiv(id, GL_INFO_LOG_LENGTH, &length);//获取需要打印的编译信息长度
        char* message = (char*)_malloca(sizeof(char) * length);
        glGetShaderInfoLog(id, length, &length, message);//获取编译失败日志
        
        std::cout << "Failed to compile " 
            << (type == GL_VERTEX_SHADER ? "vertex" : "fragment") 
            << "shader" << std::endl;
        std::cout << message << std::endl;

        glDeleteShader(id);//编译失败删除id指向的着色器
        return 0;
    }
    return id;

}
static unsigned int CreateShader(const std::string& vertexSource, const std::string& fragmentSource) {
    unsigned int program = glCreateProgram();//创建着色器程序
    unsigned int vs = CompileShader(GL_VERTEX_SHADER,vertexSource);
    unsigned int fs = CompileShader(GL_FRAGMENT_SHADER, fragmentSource);

    glAttachShader(program, vs);//将生成着色器附加到程序中
    glAttachShader(program, fs);
    glLinkProgram(program);//链接程序
    glValidateProgram(program);//验证程序的有效性

    glDeleteShader(vs);//删除已经没用的着色器
    glDeleteShader(fs);
    return program;
}
int main(){
    auto [vertexSource, fragmentSource] = ParseShader("./res/shaders/shaders.shader");
    unsigned int shader = CreateShader(vertexSource, fragmentSource);
    glUseProgram(shader);
}
```

```c++
#shader vertex
#version 330 core
layout(location = 0) in vec4 position;
void main()
{
gl_Position = position;
};

#shader fragment
#version 330 core
layout(location = 0) out vec4 color;
void main()
{
color = vec4(1.0, 0, 0, 1.0);
};
```

#### 1.2.4 顶点缓冲区（index buffer）

```cpp
unsigned int ibo;
glGenBuffers(1, &ibo);//生成bufer
unsigned int indices[] = {
	0,1,2,
    2,3,0
};
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER,ibo);//绑定当前buffer
glBufferData(GL_ELEMENT_ARRAY_BUFFER, 6 * sizeof(int), indices, GL_STATIC_DRAW);//为buffer存入数据
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, nullptr);
```

### 1.3 在OpenGL中处理错误

#### 1.3.1 glGetError

```cpp
#define ASSERT(x) if (!(x)) __debugbreak();
#define GLCall(x) GLClearError();\
        x;\
        ASSERT(GLLogCall(#x, __FILE__, __LINE__ ))
```

### 1.4 CPU传递数据给GPU

#### 1.4.1 统一变量（uniform）

统一变量是per draw，对于每个glDrawArrays，或者glDrawElements

shader文件中

```cpp
#shader fragment
#version 330 core

layout(location = 0) out vec4 color;
uniform vec4 u_Color;
void main()

{
	color = u_Color;
};
```

```cpp
int location = glUniformlocation(program, "u_Color");
glUnifore4f(location, r, g, b, a);
```

#### 1.4.2 顶点数组

```cpp
unsigned int vao;
glGenVertexArray(1,&vao);
glBindVertexArray(vao);

glVertexAttribPointer(0,.....);//0指的顶点数组中的索引0和当前顶点缓冲区绑定
```

## 2 抽象OpenGL

### 2.1 顶点缓冲区

```cpp
#pragma once
class VertexBuffer
{
private:
	unsigned int m_RenderID;
public:
	VertexBuffer(const void* data, const unsigned int size);
	~VertexBuffer();
	void Bind() const;
	void UnBind() const;
};

#include "VertexBuffer.h"
#include "Renderer.h"
VertexBuffer::VertexBuffer(const void* data, const unsigned int size)
{
	
	GLCall(glGenBuffers(1, &m_RenderID));
	GLCall(glBindBuffer(GL_ARRAY_BUFFER, m_RenderID));//绑定当前缓冲区
	GLCall(glBufferData(GL_ARRAY_BUFFER, size, data, GL_STATIC_DRAW));
}

VertexBuffer::~VertexBuffer()
{
	GLCall(glDeleteBuffers(1,&m_RenderID));
}


void VertexBuffer::Bind() const
{
	GLCall(glBindBuffer(GL_ARRAY_BUFFER, m_RenderID));//绑定当前缓冲区
}

void VertexBuffer::UnBind() const
{
	GLCall(glBindBuffer(GL_ARRAY_BUFFER, 0));//绑定当前缓冲区
}


```



### 2.2 索引缓冲区



```cpp
#pragma once
class IndexBuffer
{
private:
	unsigned int m_RenderID;
	unsigned int m_Count;
public:
	IndexBuffer(const void* data, const unsigned int count);
	~IndexBuffer();
	void Bind() const;
	void UnBind() const;
	inline unsigned int GetCount() const{
		return m_Count;
	}
};
#include "IndexBuffer.h"
#include "Renderer.h"
IndexBuffer::IndexBuffer(const void* data, const unsigned int count)
	:m_Count(count)
{
	
	GLCall(glGenBuffers(1, &m_RenderID));
	GLCall(glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, m_RenderID));//绑定当前缓冲区
	GLCall(glBufferData(GL_ELEMENT_ARRAY_BUFFER, m_Count*sizeof(unsigned int), data, GL_STATIC_DRAW));
}

IndexBuffer::~IndexBuffer()
{
	GLCall(glDeleteBuffers(1,&m_RenderID));
}


void IndexBuffer::Bind() const
{
	GLCall(glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, m_RenderID));//绑定当前缓冲区
}

void IndexBuffer::UnBind() const
{
	GLCall(glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, 0));//绑定当前缓冲区
}

```

### 2.3 顶点数组

```cpp
#pragma once
#include "VertexBuffer.h"
#include "VertexBufferLayout.h"
#include "Renderer.h"
class VertexArray
{
private:
	unsigned int m_RenderID;
public:
	void AddBufferLayout(const VertexBuffer& vb, const VertexBufferLayout& layout);
	VertexArray();
	~VertexArray();
	void Bind() const;
	void UnBind() const;

};
#include "VertexArray.h"

VertexArray::VertexArray()
{
	GLCall(glGenVertexArrays(1, &m_RenderID));
}

VertexArray::~VertexArray()
{
	GLCall(glDeleteVertexArrays(1, &m_RenderID));
}

void VertexArray::AddBufferLayout(const VertexBuffer& vb, const VertexBufferLayout& layout)
{
	Bind();
	vb.Bind();
	unsigned int offset = 0;
	const auto& elements = layout.GetElements();
	for (unsigned int i = 0; i < elements.size(); i++) {
		const auto& element = elements[i];
		GLCall(glEnableVertexAttribArray(i));
		GLCall(glVertexAttribPointer(i, element.count, element.type, element.normalized, layout.GetStride(), (const void*)offset));
		offset += element.count * VertexBufferElement::GetSizeOfType(element.type);
	}
}

void VertexArray::Bind() const
{
	GLCall(glBindVertexArray(m_RenderID));
}

void VertexArray::UnBind() const
{
	GLCall(glBindVertexArray(0));

}


```

### 2.4 布局

```cpp
#pragma once
#include <vector>
#include <GL/glew.h>
struct VertexBufferElement
{
	unsigned int type;
	unsigned int count;
	unsigned char normalized;
	static unsigned int GetSizeOfType(unsigned int type) {
		switch (type)
		{
				
		case GL_FLOAT:
			return 4;
		case  GL_UNSIGNED_INT:
			return 4;
		case GL_UNSIGNED_BYTE:
			return 1;
		}
		return 0;
	}
};
class VertexBufferLayout
{
private:
	std::vector<VertexBufferElement> elements;
	unsigned int m_Stride;
public:
	VertexBufferLayout()
        :m_Stride(0)
    {};
	~VertexBufferLayout(){};
	template<typename T>
	void Push(unsigned int count) {
		static_assert(false);
	}
	template<>
	void Push<float>(unsigned int count) {
		elements.push_back({GL_FLOAT, count,GL_FALSE });
		m_Stride += count * VertexBufferElement::GetSizeOfType(GL_FLOAT);
	}	

	template<>
	void Push<unsigned int>(unsigned int count) {
		elements.push_back({ GL_UNSIGNED_INT, count, GL_FALSE });
		m_Stride += count * VertexBufferElement::GetSizeOfType(GL_UNSIGNED_INT);
	}

	template<>
	void Push<unsigned char>(unsigned int count) {
		elements.push_back({ GL_UNSIGNED_BYTE, count, GL_TRUE });
		m_Stride += count * VertexBufferElement::GetSizeOfType(GL_UNSIGNED_BYTE);
	}

	inline std::vector<VertexBufferElement> GetElements() const {
		return elements;
	}
	inline unsigned int GetStride()const {
		return m_Stride;
	}
};


```

### 2.5 着色器

```cpp
#pragma once
#include <unordered_map>
#include<iostream>
#include<fstream>
#include<sstream>
#include<tuple>


class Shader
{
private:
	unsigned int m_RenderID;
	std::string m_FliePath;
	std::unordered_map<std::string, int> m_UniformLocationCache;
public:
	Shader(const std::string& filepath);
	~Shader();
	void Bind() const;
	void UnBind() const;
	void SetUniform4f(const std::string& name, float v0, float v1, float v2, float v3);
private:
	std::tuple<std::string, std::string> ParseShader(const std::string& filepath);
	unsigned int CompileShader(unsigned int type, const std::string& source);
	unsigned int CreateShader(const std::string& vertexSource, const std::string& fragmentSource);

	int GetUniformLocation(const std::string& name);
};

#include "Shader.h"
#include "Renderer.h"
Shader::Shader(const std::string& filepath)
	:m_FliePath(filepath)
{
	auto [vertexSource, fragmentSource] = ParseShader(filepath);
	m_RenderID = CreateShader(vertexSource, fragmentSource);
}

Shader::~Shader()
{
	GLCall(glDeleteProgram(m_RenderID));
}

void Shader::Bind() const
{
	GLCall(glUseProgram(m_RenderID));
}

void Shader::UnBind() const
{
	GLCall(glUseProgram(0));
}


std::tuple<std::string, std::string> Shader::ParseShader(const std::string& filepath) {
	std::ifstream stream(filepath);
	std::string line;
	std::stringstream ss[2];
	enum class ShaderType {
		None = -1,
		Vertex = 0,
		Fragment = 1
	};
	ShaderType type = ShaderType::None;
	while (getline(stream, line))
	{
		if (line.find("#shader") != std::string::npos) {
			if (line.find("vertex") != std::string::npos)
				type = ShaderType::Vertex;
			else
				type = ShaderType::Fragment;
		}
		else {
			ss[(int)type] << line << "\n";
		}
	}
	return std::tuple<std::string, std::string>(ss[0].str(), ss[1].str());

}
/*
* 编译着色器
*/
unsigned int Shader::CompileShader(unsigned int type, const std::string& source) {
	unsigned int id = glCreateShader(type);
	const char* src = source.c_str();
	glShaderSource(id, 1, &src, nullptr);//绑定着色器代码，生成着色器
	glCompileShader(id);//编译id表示的着色器

	int result;
	glGetShaderiv(id, GL_COMPILE_STATUS, &result);//获取当前编译状态
	if (result == GL_FALSE) {
		int length;
		glGetShaderiv(id, GL_INFO_LOG_LENGTH, &length);//获取需要打印的编译信息长度
		char* message = (char*)_malloca(sizeof(char) * length);
		glGetShaderInfoLog(id, length, &length, message);//获取编译失败日志

		std::cout << "Failed to compile "
			<< (type == GL_VERTEX_SHADER ? "vertex" : "fragment")
			<< "shader" << std::endl;
		std::cout << message << std::endl;

		glDeleteShader(id);//编译失败删除id指向的着色器
		return 0;
	}
	return id;

}
unsigned int Shader::CreateShader(const std::string& vertexSource, const std::string& fragmentSource) {
	unsigned int program = glCreateProgram();//创建着色器程序
	unsigned int vs = CompileShader(GL_VERTEX_SHADER, vertexSource);
	unsigned int fs = CompileShader(GL_FRAGMENT_SHADER, fragmentSource);

	glAttachShader(program, vs);//将生成着色器附加到程序中
	glAttachShader(program, fs);
	glLinkProgram(program);//链接程序
	glValidateProgram(program);//验证程序的有效性

	glDeleteShader(vs);//删除已经没用的着色器
	glDeleteShader(fs);
	return program;
}

int Shader::GetUniformLocation(const std::string& name)
{
	if (m_UniformLocationCache.find(name) != m_UniformLocationCache.end())
		return m_UniformLocationCache[name];
	GLCall(int location = glGetUniformLocation(m_RenderID, name.c_str()));
	if (location == -1)
		std::cout << "warning:uniform" << name << "does not exist!" << std::endl;
	m_UniformLocationCache[name] = location;
	return location;

}

void Shader::SetUniform4f(const std::string& name, float v0, float v1, float v2, float v3)
{
	GLCall(glUniform4f(GetUniformLocation(name),v0,v1,v2,v3));
}

```



### 2.6 渲染器

```cpp
#pragma once
#include<gl/glew.h>
#include "VertexArray.h"
#include "IndexBuffer.h"
#include "Shader.h"

#define ASSERT(x) if (!(x)) __debugbreak();
#define GLCall(x) GLClearError();\
        x;\
        ASSERT(GLLogCall(#x, __FILE__, __LINE__ ))
 void GLClearError(); 
 bool GLLogCall(const char* function, const char* file, int line); 
 class Renderer
 {
 public:
	 Renderer();
	 ~Renderer();
	 void Clear() const;
	 void Draw(const VertexArray& va, const IndexBuffer& ib, const Shader& shader);

 };
#include "Renderer.h"
#include<iostream>

void GLClearError()
{
    while (glGetError() != GL_NO_ERROR);
}
bool GLLogCall(const char* function, const char* file, int line)
{
    while (GLenum error = glGetError())
    {
        std::cout << "[OpenGL Error] (" << error << "):"
            << function << " " << file << " " << line << std::endl;
        return false;
    }
    return true;
}

Renderer::Renderer()
{

}

Renderer::~Renderer()
{

}

void Renderer::Clear() const
{
    GLCall(glClear(GL_COLOR_BUFFER_BIT));
}

void Renderer::Draw(const VertexArray& va, const IndexBuffer& ib, const Shader& shader)
{
    shader.Bind();
    va.Bind();
    ib.Bind();
    GLCall(glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, nullptr));
}


```



## 3 纹理与混合

### 3.1 Texture

#### 3.1.1 TextureClass

```cpp
#pragma once
#include "Renderer.h"
class Texture
{
private:
	unsigned int m_RenderID;
	std::string m_FilePath;
	unsigned char* m_LocalBuffer;
	int m_Width, m_Height, m_BPP;//bits per pixel
public:
	Texture(const std::string& path);
	~Texture();
	void Bind(unsigned int slot = 0) const;
	void UnBind() const;
};
#include "Texture.h"
#include "stb_image/stb_image.h"
Texture::Texture(const std::string& path)
	:m_RenderID(0),m_FilePath(path), m_LocalBuffer(nullptr),
	m_Width(0),m_Height(0),m_BPP(0)
{
	stbi_set_flip_vertically_on_load(1);
	m_LocalBuffer = stbi_load(m_FilePath.c_str(), &m_Width, &m_Height, &m_BPP, 4);

	GLCall(glGenTextures(1, &m_RenderID));
	GLCall(glBindTexture(GL_TEXTURE_2D, m_RenderID));

	GLCall(glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR));
	GLCall(glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR));
	GLCall(glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE));
	GLCall(glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE));

	GLCall(glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA8, m_Width, m_Height, 0, GL_RGBA, GL_UNSIGNED_BYTE, m_LocalBuffer));
	GLCall(glBindTexture(GL_TEXTURE_2D, 0));
}

Texture::~Texture()
{
	GLCall(glDeleteTextures(1, &m_RenderID));
}

void Texture::Bind(unsigned int slot /*= 0*/) const
{
	GLCall(glActiveTexture(GL_TEXTURE0 + slot));
	GLCall(glBindTexture(GL_TEXTURE_2D, m_RenderID));
}

void Texture::UnBind() const
{
	GLCall(glBindTexture(GL_TEXTURE_2D, 0));
}

```

#### 3.1.2 着色器修改

```cpp
layout(location = 1) in vec2 texCoord;

out vec2 v_TexCoord;

#shader fragment
#version 330 core

layout(location = 0) out vec4 color;

uniform vec4 u_Color;
uniform sampler2D u_Texture;

in vec2 v_TexCoord;
void main()
{
	vec4 texColor = texture(u_Texture, v_TexCoord);
	color = texColor;
	
};

```



