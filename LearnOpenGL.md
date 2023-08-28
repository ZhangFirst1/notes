## LearnOpenGL

### 1. 画两个三角形拼成的矩形

```
#include <glad/glad.h>
#include <GLFW/glfw3.h>
#include <iostream>

void framebuffer_size_callback(GLFWwindow* window, int width, int height);
void processInput(GLFWwindow* window);

const unsigned int SCR_WIDTH = 800;
const unsigned int SCR_HEIGHT = 600;

const char* vertexShaderSource = "#version 330 core\n"
"layout (location = 0) in vec3 aPos;\n"
"void main()\n"
"{\n"
"	gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n"
"}\0";
const char* fragmentShaderSource = "#version 330 core\n"
"out vec4 FragColor;\n"
"void main()\n"
"{\n"
"	FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);\n"
"}\n\0";

int main() {
	//glfw: initialize and configure
	glfwInit();
	glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
	glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
	glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

#ifdef __APPLE__
	glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);
#endif

	// glfw window creation
	GLFWwindow* window = glfwCreateWindow(SCR_WIDTH, SCR_HEIGHT, "learnOpenGL", NULL, NULL);
	if (window == NULL) {
		std::cout << "Failed to creat GLFW window" << std::endl;
		glfwTerminate();
		return -1;
	}
	glfwMakeContextCurrent(window);
	// 注册回调函数
	glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);

	// 使用glad加载OpenGL函数指针
	// ----------------------------------
	if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress)) {
		std::cout << "Failed to initialize GLAD" << std::endl;
		return -1;
	}

	// build and compile our shader program
	// ------------------------------------
	// vertex shader
	unsigned int vertexShader = glCreateShader(GL_VERTEX_SHADER);
	// 把着色器源码附着到着色器对象上并编译
	// 着色器对象 源码字符串数量，源码，
	glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
	glCompileShader(vertexShader);
	// 检查着色器编译错误
	int success;
	char infoLog[512];
	glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);
	if (!success) {
		glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);
		std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;
	}
	// fragment shader
	unsigned int fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
	glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);
	glCompileShader(fragmentShader);
	glGetShaderiv(fragmentShader, GL_COMPILE_STATUS, &success);
	if (!success) {
		glGetShaderInfoLog(fragmentShader, 512, NULL, infoLog);
		std::cout << "ERROR::SHADER::FRAGMENT::COMPILATION_FAILED\n" << infoLog << std::endl;
	}
	// link shaders
	unsigned int shaderProgram = glCreateProgram();
	glAttachShader(shaderProgram, vertexShader);
	glAttachShader(shaderProgram, fragmentShader);
	glLinkProgram(shaderProgram);
	// check link error
	glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);
	if (!success) {
		glGetProgramInfoLog(shaderProgram, 512, NULL, infoLog);
		std::cout << "ERROR::SHADER::PROGRAM::LINKING_FAILED\n" << infoLog << std::endl;
	}
	glDeleteShader(vertexShader);
	glDeleteShader(fragmentShader);

	// 设置顶点数据和属性
	float vertices[] = {
			0.5f,  0.5f, 0.0f,  // top right
			0.5f, -0.5f, 0.0f,  // bottom right
		   -0.5f, -0.5f, 0.0f,  // bottom left
		   -0.5f,  0.5f, 0.0f   // top left 
	};
	unsigned int indices[] = {  // note that we start from 0!
		0, 1, 3,  // first Triangle
		1, 2, 3   // second Triangle
	};

	unsigned int VBO, VAO, EBO;
	glGenVertexArrays(1, &VAO);
	glGenBuffers(1, &VBO);
	glGenBuffers(1, &EBO);
	// 先绑定VAO，再绑定设置VBO，最后设置顶点属性
	glBindVertexArray(VAO);

	glBindBuffer(GL_ARRAY_BUFFER, VBO);
	// 缓冲类型 传输数据的大小 实际数据 如何管理数据（三角形位置数据几乎不改变，用GL_STATIC_DRAW）
	glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

	glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
	glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);
	// 使用glVertexAttribPointer告诉OpenGL如何解析顶点数据
	// 顶点属性 顶点属性大小 指定数据类型 数据是否被标准化 步长 数据在缓冲中起始位置偏移量
	glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
	glEnableVertexAttribArray(0);	//顶点属性位置值作为参数，启用顶点属性

	// 解绑VAO和VBO
	glBindBuffer(GL_ARRAY_BUFFER, 0);
	glBindVertexArray(0);

	glPolygonMode(GL_FRONT_AND_BACK, GL_LINE);
	//render loop
	// -------------------------------------
	while (!glfwWindowShouldClose(window)) {
		// input
		// ------
		processInput(window);

		// render
		// -------
		glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
		glClear(GL_COLOR_BUFFER_BIT);

		// draw triangle
		glUseProgram(shaderProgram);
		glBindVertexArray(VAO);
		//glDrawArrays(GL_TRIANGLES, 0, 3);
		glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
		// swap buffers and poll IO events
		glfwSwapBuffers(window);
		glfwPollEvents();
	}
	// de-allocate all resources
	// ---------
	glDeleteVertexArrays(1, &VAO);
	glDeleteBuffers(1, &VBO);
	glDeleteBuffers(1, &EBO);
	glDeleteProgram(shaderProgram);

	glfwTerminate();
	return 0;
}

void framebuffer_size_callback(GLFWwindow* window, int width, int height) {
	glViewport(0, 0, width, height);
}
//exit
void processInput(GLFWwindow* window) {
	if (glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
		glfwSetWindowShouldClose(window, true);
}
```

[OpenGL学习笔记（三）绘制三角形/四边形 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/107883746)

**绘制三角形流程**

**1. 定义顶点数据数组**

**2. 创建，绑定VAO，管理顶点属性**

1. 创建VAO；glGenVertexArrays(1, &VAO);
2. 绑定VAO；glBindVertexArray(VAO);

**3.创建，绑定VBO，管理顶点数据储存的内存**

1. 创建VBO；glGenBuffer(1, &VBO);
2. 绑定VBO； glBindBuffer(GL_ARRAY_BUFFER, VBO);
3. 把顶点数据拷贝到VBO缓冲内存中；glBufferData();

**4.创建，绑定EBO 管理顶点数据存储的内存**

1. 创建EBO；glGenBuffers(1, &EBO);
2. 绑定EBO；glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);

**5.链接顶点属性 设置顶点属性指针**

1. 告诉顶点着色器如何解析顶点数据；glVertexAttribPointer(..);
2. 启用顶点属性；glEnableVertexAttribArray(0);

**6.定义顶点着色器**

1. 声明所用的OpenGL版本号及模块；#Version 330 core;
2. 声明用来接受顶点数据的遍量及位置；in vec3 aPos;
3. 复制；预定义变量gl_Positionl;

**7.编译顶点着色器**

1. 创建顶点着色器对象；glCreateShader(CL_VERTEX_SHADER)；
2. 把定义的着色器源码附加到顶点着色器对象上；glShaderSource(...)；
3. 编译着色器；glCompileShader(VertexShader)；
4. 检查是否编译成功，并打印信息；glGetShaderriv()；

**8.定义片元着色器**

1. 声明所用的OpenGL版本号以及模块；#Version 330 core；
2. 声明用来输出颜色的变量；out vec4 FragColor；
3. 简单的赋值；预定义变量FragColor ；

**9.编译片元着色器**

1. 创建片源着色器对象；glCreateShader(GL_FRAGMENT_SHADER)；
2. 把定义片源着色器源码附加到片源着色器对象上；glShaderSource(...)；
3. 编译着色器；glCompileShader(fragmentShader)；
4. 检查是否编译成功，并打印信息；glGetShaderriv()；

**10.链接着色器程序**

1. 创建链接着色程序；glCreateProgram()；
2. 把之前的顶点与片源绑定到链接程序上；glAttachShader(..)；
3. 链接编译；glLinkProgram(shaderProgram)；
4. 检查链接编译是否成功，并打印信息；glGetProgramInfoLog(...)；
5. 激活程序对象(一般放在渲染循环里)；glUseProgram(shaderProgram)；
6. 渲染绘制；glDrawArrays(...);
7. 删除创建的着色器；glDeleteShader(vertexShader/fragmentShader)；