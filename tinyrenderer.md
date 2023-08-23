## tinyrenderer

​	跟着[ssloy/tinyrenderer: A brief computer graphics / rendering course (github.com)](https://github.com/ssloy/tinyrenderer)项目实现一个基础的渲染器。

![image-20230807160409020](D:\typora\图片\image-20230807160409020.png)

### Lesson 1: 画直线

```cpp
void line(int x0, int y0, int x1, int y1, TGAImage& image, TGAColor color) {
	bool steep = false;
	if (std::abs(x0 - x1) < std::abs(y0 - y1)) {	//直线斜率大于1
		//对调端点(x,y)坐标，使得斜率小于1，不会产生y的间隔
		std::swap(x0, y0);
		std::swap(x1, y1);
		steep = true;
	}

	if (x0 > x1) {	//保证x从小到大
		std::swap(x0, x1);
		std::swap(y0, y1);
	}

	int dx = x1 - x0;
	int dy = y1 - y0;
	//float derror = std::abs(dy / float(dx));	//使用累计误差的方法优化
	int derror2 = std::abs(dy) * 2;			//两边*2 等式变形 无需使用除法和浮点数
	int error2 = 0;
	int y = y0;
	for (float x = x0; x <= x1; x++) {
		//float t = (x - x0) / (float)(x1 - x0);
		//int y = y0 * (1.0 - t) + y1 * t;
		if (steep == true) {
			image.set(y, x, color);
		}
		else {
			image.set(x, y, color);
		}
		error2 += derror2;
		if (error2 > dx) {
			y += (y1 > y0 ? 1 : -1);
			error2 -= dx * 2;
		}
	}
}
```

### Lesson 2: 画线框

​	使用.obj格式文件

**1.v顶点（Geometric vertices）**

```text
v 1.000000 -1.000000 -1.000000 
```

- 格式： **vt u v w**

- 意义：表示顶点，每个顶点的坐标

**2. vt 顶点纹理坐标 (Texture vertices)**

```text
vt 0.748573 0.750412 
vt 0.749279 0.501284 
vt 0.999110 0.501077
```

- 格式：vt u v w
- 意义：绘制模型的三角面片时，每个顶点取像素点时对应的纹理图片上的坐标。纹理图片的坐标指的是，纹理图片如果被放在屏幕上显示时，以屏幕左下角为原点的坐标。
- 注意：w 一般用于形容三维纹理，大部分是用不到的，基本都为 0

**3. vn 顶点法向量 (Vertex normals)**

```text
vn 0.000000 0.000000 -1.000000 
vn -1.000000 -0.000000 -0.000000 
vn -0.000000 -0.000000 1.000000
```

- 格式：vn x y z
- 意义：绘制模型三角面片时，需要确定三角面片的朝向，整个面的朝向，是由构成每个面的顶点对应的顶点法向量的做矢量和决定的（xyz 的坐标分别相加再除以 3 得到的）。

**4. f 面 (Face)**

```text
f 5/1/1 1/2/1 4/3/1 
f 5/1/1 4/3/1 8/4/1 
f 3/5/2 7/6/2 8/7/2
```

- 格式 ：f v/vt/vn v/vt/vn v/vt/vn（f 顶点索引 / 纹理坐标索引 / 顶点法向量索引）

**测试代码：**

```cpp
	if (2 == argc) {
		model = new Model(argv[1]);	//命令行方式构造
	}
	else {
		model = new Model("obj/african_head.obj");	//代码构造
	}

	TGAImage image(width, height, TGAImage::RGB);
	for (int i = 0; i < model->nfaces(); i++) {
		std::vector<int> face = model->face(i);	//保存一个面的三个顶点坐标
		for (int j = 0; j < 3; j++) {
			//根据两个顶点划线
			Vec3f v0 = model->vert(face[j]);
			Vec3f v1 = model->vert(face[(j + 1) % 3]);
			//模型坐标 -> 屏幕坐标
			//[-1,1] -> [0,2] -> [0,800]
			int x0 = (v0.x + 1.) * width / 2.;	
			int y0 = (v0.y + 1.) * height / 2.;
			int x1 = (v1.x + 1.) * width / 2.;
			int y1 = (v1.y + 1.) * height / 2.;
			line(x0, y0, x1, y1, image, white);
		}
	}

	image.flip_vertically(); // i want to have the origin at the left bottom corner of the image
	image.write_tga_file("output.tga");
	delete model;
```

### Lesson 3: 三角形光栅化和Z-buffer

**逐行画法**(需要重载上方的line适配Vec2i)

```cpp
/*使用逐行画法*/
void triangle(Vec2i t0, Vec2i t1, Vec2i t2, TGAImage& image, TGAColor color) {
	if (t0.y == t1.y && t1.y == t2.y) return;	//三角形面积为零
	//保证t0->t2是从底到上
	if (t0.y > t1.y) std::swap(t0, t1);
	if (t0.y > t2.y) std::swap(t0, t2);
	if (t1.y > t2.y) std::swap(t1, t2);
	//以y轴为差填充三角形
	int tall = t2.y - t0.y;
	for (int i = 0; i < tall; i++) {
		//根据t1将三角形分成上下两部分
		bool second_half = i > t1.y - t0.y || t1.y == t0.y;	//上部为1 下部为0
		int segment_height = second_half ? t2.y - t1.y : t1.y - t0.y; //上下部高度
		float alpha = (float)i / tall;
		float beta = (float)(i - (second_half ? t1.y - t0.y : 0)) / segment_height;
		//计算左右两点坐标
		Vec2i A = t0 + (t2 - t0) * alpha;
		Vec2i B = second_half ? t1 + (t2 - t1) * beta : t0 + (t1 - t0) * beta;
		if (A.x > B.x) std::swap(A, B);
		//根据A和B对当前高度着色
		for (int j = A.x; j <= B.x; j++) {
			image.set(j, t0.y + i, color);
		}
	}
}
```

**bounding box**

```cpp
/*
	使用重心坐标判断三角形bounding box内像素是否在三角形内绘制三角形
*/
Vec3f barycentric(Vec3f A, Vec3f B, Vec3f C, Vec3f P) {
	Vec3f s[2];
	for (int i = 2; i--; ) {
		s[i][0] = C[i] - A[i];
		s[i][1] = B[i] - A[i];
		s[i][2] = A[i] - P[i];
	}
	Vec3f u = cross(s[0], s[1]);
	//z分量为零 不在三角形内
	if (std::abs(u[2]) > 1e-2) return Vec3f(1.f - (u.x + u.y) / u.z, u.y / u.z, u.x / u.z);
	//返回P点重心坐标
	return Vec3f(-1, 1, 1);
}

//重载triangle
void triangle(Vec3f* pts,float *zbuffer, TGAImage& image, TGAColor color) {
	Vec2f bboxmin(std::numeric_limits<float>::max(), std::numeric_limits<float>::max());
	Vec2f bboxmax(-std::numeric_limits<float>::max(), -std::numeric_limits<float>::max());
	Vec2f clamp(image.get_width() - 1, image.get_height() - 1);
	//确定bounding box 大小
	for (int i = 0; i < 3; i++) {
		for (int j = 0; j < 2; j++) {
			bboxmin[j] = std::max(0.f, std::min(bboxmin[j], pts[i][j]));
			bboxmax[j] = std::min(clamp[j], std::max(bboxmax[j], pts[i][j]));
		}
	}
	Vec3f P;
	for (P.x = bboxmin.x; P.x <= bboxmax.x; P.x++)
		for (P.y = bboxmin.y; P.y <= bboxmax.y; P.y++) {
			Vec3f screen = barycentric(pts[0], pts[1], pts[2], P);
			//三个分量之一小于零说明像素在三角形外
			if (screen.x < 0 || screen.y < 0 || screen.z < 0) continue;
			P.z = 0;
			//计算zbuffer 并 每个顶点的z值乘上对应的质心坐标分量
			for (int i = 0; i < 3; i++) P.z += pts[i][2] * screen[i];
			if (zbuffer[int(P.x + P.y * width)] < P.z) {
				zbuffer[int(P.x + P.y * width)] = P.z;
				image.set(P.x, P.y, color);
			}
		}
}
```

```cpp
if (2 == argc) {
		model = new Model(argv[1]);	//命令行方式构造
	}
	else {
		model = new Model("obj/african_head.obj");	//代码构造
	}
	TGAImage image(width, height, TGAImage::RGB);
	//创建zbuffer并初始化为一个很小的值
	float* zbuffer = new float[width * height];
	for (int i = width * height; i--; zbuffer[i] = -std::numeric_limits<float>::max());

	Vec3f light_dir(0, 0, -1); // define light_dir
	for (int i = 0; i < model->nfaces(); i++) {
		std::vector<int> face = model->face(i);
		//屏幕坐标与世界坐标
		Vec3f world_coords[3];
		for (int j = 0; j < 3; j++) {
			world_coords[j] = model->vert(face[j]);
		}
		//法线方向 并 正交化
		Vec3f n = cross((world_coords[2] - world_coords[0]), (world_coords[1] - world_coords[0]));
		n.normalize();
		//光照强度=法向量*光线方向（点乘）
		float intensity = n * light_dir;
		if (intensity > 0) {
			Vec3f pts[3];
			for (int j = 0; j < 3; j++) pts[j] = world2screen(model->vert(face[j]));
			triangle(pts, zbuffer, image, TGAColor(intensity * 255, intensity * 255, intensity * 255, 255));
			//triangle(pts, zbuffer, image, TGAColor(rand() % 255, rand() % 255, rand() % 255, 255));
		}
	}
	image.flip_vertically(); // i want to have the origin at the left bottom corner of the image
	image.write_tga_file("output3.tga");
	delete model;
```

### Lesson 4 & 5: 变换

[3D物体渲染到2D屏幕的矩阵变换过程：模型变换（Modeling Trans）、视图变换(View Trans)和投影变换(Projection Trans) - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/466508365)

 ```
 Viewport * Projection * View * Model * v.
 
 Vec3f v = model->vert(face[j]);
 screen_coords[j] =  Vec3f(ViewPort*Projection*ModelView*Matrix(v));
 ```

1.首先将模型三角形平面的三个顶点向量转为矩阵

```
Matrix v2m(Vec3f v) {
	Matrix m(4, 1);
    m[0][0] = v.x;
    m[1][0] = v.y;
    m[2][0] = v.z;
    m[3][0] = 1.f;
    return m;
}
```

2.ModelView（视图变换）变换，改变相机位置与角度

```
Matrix lookat(Vec3f eye, Vec3f center, Vec3f up) {
    Vec3f z = (eye - center).normalize();
    Vec3f x = (up ^ z).normalize();
    Vec3f y = (z ^ x).normalize();
    Matrix rotation = Matrix::identity(4);
    Matrix translation = Matrix::identity(4);

    for (int i = 0; i < 3; i++) {
        translation[i][3] = -center[i];
    }

    for (int i = 0; i < 3; i++) {
        rotation[0][i] = x[i];
        rotation[1][i] = y[i];
        rotation[2][i] = z[i];
    }
    return rotation * translation;
}

Matrix ModelView = lookat(eye, center, Vec3f(0,1,0));
```

3.Projection（投影变换）变换，产生透视投影效果

```
Matrix Projection = Matrix::identity(4);
Projection[3][2] = -1.f / (eye - center).norm();
```

4.ViewPort（视口变换）变换，将[-1,1]^[-1,1]的世界坐标转换为屏幕坐标[w/2, h/2, depth/2]，保留深度信息用以做z-buffer

```
Matrix viewport(int x, int y, int w, int h)
{
    Matrix m = Matrix::identity(4);
    m[0][3] = x + w / 2.f;
    m[1][3] = y + h / 2.f;
    m[2][3] = depth / 2.f;

    m[0][0] = w / 2.f;
    m[1][1] = h / 2.f;
    m[2][2] = depth / 2.f;
    return m;
}
```

5.将矩阵转为向量方便进行光栅化

```
Vec3f m2v(Matrix m) {
	return Vec3f(m[0][0] / m[3][0], m[1][0] / m[3][0], m[2][0] / m[3][0]);
}
```

补充：应用纹理坐标后需要进行插值

```
Vec3i   P = Vec3f(A) + Vec3f(B - A) * phi;	//点插值
Vec2i uvP = uvA + (uvB - uvA) * phi;		//纹理坐标插值
float ityP = ityA + (ityB - ityA) * phi;	//intensity插值
```

**与games101中管线的异同之处**

[3D物体渲染到2D屏幕的矩阵变换过程：模型变换（Modeling Trans）、视图变换(View Trans)和投影变换(Projection Trans) - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/466508365)

1. 并没有实现MVP变换中的Model变换（调整模型位置）。
2. 视图变换与之相同。
3. games101中，实现Projection是乘上p矩阵，即把视锥远裁剪面缩小到与近裁剪面相同大小，再按照正交投影将包裹所有可见模型物体的虚拟长方体转换到取值[-1,1] [-1,1]的2 * 2 * 2的立方体中（长方体转为立方体是为了方便后续的裁剪以及其他操作的计算）。实际投影矩阵是GPU自己做了透视除法，而项目中只自己实现了透视除法。
4. games101中，近平面是自定义的，且投影到近平面上，而项目中默认是投影到xoy面（计算时不区分近远平面）。
5. 视口变换与games101中相同。

## Lesson 6 & 7: 渲染 & 阴影

​	按照之前实现的代码十分混乱，让我们来重构代码。

分为：geometry.h + .cpp

​	 	   model.h + .cpp

​			our_gl.h + .cpp

​			main.cpp

**geometry:** 使用模板元编程实现Vec，mat及相关操作。

**model:** 定义模型，以及模型相关操作。

```cpp
class Model {
private:
	std::vector<Vec3f> verts_;		//顶点
	std::vector<std::vector<Vec3i> > faces_;	// vertex/uv/normal index
	std::vector<Vec3f> norms_;		//法线
	std::vector<Vec2f> uv_;			//纹理uv坐标
	//Bling-Phong
	TGAImage diffusemap_;			//纹理图（也就是diffuse）
	TGAImage normalmap_;			//法线贴图
	TGAImage specularmap_;			//高光图（用于补高光）
	//加载材质
	void load_texture(std::string filename, const char* suffixl, TGAImage& img);
public:
	Model(const char* filename);
	~Model();
	int nverts();							//面和点的总数量
	int nfaces();
	Vec3f normal(int iface, int nthvert);	//返回方向向量
	Vec3f normal(Vec2f uv);
	Vec3f vert(int i);						//返回指定点坐标
	Vec3f vert(int iface, int nthvert);		
	Vec2f uv(int iface, int nthvert);		//返回uv坐标
	TGAColor diffuse(Vec2f uv);				//纹理
	float specular(Vec2f uv);				//高光
	std::vector<int> face(int idx);			//返回三角形的三个顶点坐标
};

```

**our_gl: **定义IShader类和triangle方法，IShader虚类中有纯虚函数**顶点着色器**和**片元着色器**，其中顶点着色器用于计算颜色和计算变换矩阵，片元着色器用于计算法线、光照，应用纹理。

```
extern Matrix ModelView;
extern Matrix ViewPort;
extern Matrix Projection;
const float depth = 2000.f;

void viewport(int x, int y, int w, int h);
void projection(float coeff = 0.f);
void lookat(Vec3f eye, Vec3f center, Vec3f up);

//抽象类 
struct IShader {
	virtual ~IShader();
	//纯虚函数 顶点着色器 和 片元着色器必须实现
	virtual Vec4f vertex(int iface, int nthvert) = 0;
	virtual bool fragment(Vec3f bar, TGAColor& color) = 0;
};

void triangle(Vec4f* pts, IShader& shader, TGAImage& image, float* zbuffer);
```

```cpp
Matrix ModelView;
Matrix ViewPort;
Matrix Projection;

IShader::~IShader() {}
//视口变换
void viewport(int x, int y, int w, int h) {
	ViewPort = Matrix::identity();
	ViewPort[0][3] = x + w / 2.f;
	ViewPort[1][3] = y + h / 2.f;
	ViewPort[2][3] = 255.f / 2.f;

	ViewPort[0][0] = w / 2.f;
	ViewPort[1][1] = h / 2.f;
	ViewPort[2][2] = 255.f / 2.f;
}
//投影变换
void projection(float coeff) {
	Projection = Matrix::identity();
	Projection[3][2] = coeff;
}
//视图变换
void lookat(Vec3f eye, Vec3f center, Vec3f up) {
	Vec3f z = (eye - center).normalize();
	Vec3f x = cross(up, z).normalize();
	Vec3f y = cross(z, x).normalize();
	ModelView = Matrix::identity();
	for (int i = 0; i < 3; i++) {
		ModelView[0][i] = x[i];
		ModelView[1][i] = y[i];
		ModelView[2][i] = z[i];
		ModelView[i][3] = -center[i];
	}
}
//重心坐标计算
Vec3f barycentric(Vec2f A, Vec2f B, Vec2f C, Vec2f P) {
	Vec3f s[2];
	for (int i = 2; i--; ) {
		s[i][0] = C[i] - A[i];
		s[i][1] = B[i] - A[i];
		s[i][2] = A[i] - P[i];
	}
	Vec3f u = cross(s[0], s[1]);
	if (std::abs(u[2]) > 1e-2) // dont forget that u[2] is integer. If it is zero then triangle ABC is degenerate
		return Vec3f(1.f - (u.x + u.y) / u.z, u.y / u.z, u.x / u.z);
	return Vec3f(-1, 1, 1); // in this case generate negative coordinates, it will be thrown away by the rasterizator
}
//填充三角形
//输入 齐次坐标 着色器 图像 z-buffer
void triangle(Vec4f* pts, IShader& shader, TGAImage& image, float* zbuffer) {
	//bounding box
	Vec2f bboxmin(std::numeric_limits<float>::max(), std::numeric_limits<float>::max());
	Vec2f bboxmax(-std::numeric_limits<float>::max(), -std::numeric_limits<float>::max());
	for (int i = 0; i < 3; i++) {
		for (int j = 0; j < 2; j++) {
			bboxmin[j] = std::min(bboxmin[j], pts[i][j] / pts[i][3]);
			bboxmax[j] = std::max(bboxmax[j], pts[i][j] / pts[i][3]);
		}
	}
	Vec2i P;
	TGAColor color;
	for (P.x = bboxmin.x; P.x <= bboxmax.x; P.x++) {
		for (P.y = bboxmin.y; P.y <= bboxmax.y; P.y++) {
			//重心坐标
			Vec3f c = barycentric(proj<2>(pts[0] / pts[0][3]), proj<2>(pts[1] / pts[1][3]), proj<2>(pts[2] / pts[2][3]), proj<2>(P));
			//深度插值
			float z = pts[0][2] * c.x + pts[1][2] * c.y + pts[2][2] * c.z;
			float w = pts[0][3] * c.x + pts[1][3] * c.y + pts[2][3] * c.z;
			int frag_depth = z / w;
			//不在三角形内或深度测试失败
			if (c.x < 0 || c.y < 0 || c.z<0 || zbuffer[P.x + P.y * image.get_width()]>frag_depth) continue;
			//片元着色器判断是否要丢弃该像素点（默认不丢弃）
			bool discard = shader.fragment(c, color);
			if (!discard) {
				//更新深度数组并填充像素
				zbuffer[P.x + P.y * image.get_width()] = frag_depth;
				image.set(P.x, P.y, color);
			}
		}
	}
}
```

**main: **实现继承自IShader类的Shader类，并实现顶点和片元着色器。

​			实现继承自IShader类的DepthShader类，用于深度检测，渲染两次，第一次深度测试，第二次着色。

```
#include <vector>
#include <iostream>

#include "tgaimage.h"
#include "model.h"
#include "geometry.h"
#include "our_gl.h"

Model* model = NULL;
float* shadowbuffer = NULL;

const int width = 800;
const int height = 800;

Vec3f light_dir(1, 1, 1);
Vec3f       eye(1, 1, 3);
Vec3f    center(0, 0, 0);
Vec3f        up(0, 1, 0);

struct Shader : public IShader {
    mat<4, 4, float> uniform_M;         //  Projection*ModelView
    mat<4, 4, float> uniform_MIT;       // (Projection*ModelView).invert_transpose()
    mat<4, 4, float> uniform_Mshadow;   // 当前片段的屏幕坐标转换为阴影缓冲区内的屏幕坐标
    mat<2, 3, float> varying_uv;        // uv坐标齐次坐标, VS中写，FS中读
    mat<3, 3, float> varying_tri;       // 视口变换前的三角形齐次坐标，VS中写，FS中读

    Shader(Matrix M, Matrix MIT, Matrix MS) : uniform_M(M), uniform_MIT(MIT), uniform_Mshadow(MS), varying_uv(), varying_tri() {}

    virtual Vec4f vertex(int iface, int nthvert) {
        Vec4f gl_Vertex = ViewPort * Projection * ModelView * embed<4>(model->vert(iface, nthvert));    //变换矩阵
        varying_uv.set_col(nthvert, model->uv(iface, nthvert));             //设置uv坐标
        varying_tri.set_col(nthvert, proj<3>(gl_Vertex / gl_Vertex[3]));    //视口变换前的三角形齐次坐标
        return gl_Vertex;
    }

    virtual bool fragment(Vec3f bar, TGAColor& color) {
        Vec4f sb_p = uniform_Mshadow * embed<4>(varying_tri * bar);     // corresponding point in the shadow buffer
        sb_p = sb_p / sb_p[3];
        int idx = int(sb_p[0]) + int(sb_p[1]) * width;                  // 深度检测数组下标
        float shadow = .3 + .7 * (shadowbuffer[idx] < sb_p[2] + 43.34); // 解决z-fighting问题
        Vec2f uv = varying_uv * bar;                                    // 对当前像素插值uv坐标
        Vec3f n = proj<3>(uniform_MIT * embed<4>(model->normal(uv))).normalize(); // 法线
        Vec3f l = proj<3>(uniform_M * embed<4>(light_dir)).normalize(); // 入射光线
        Vec3f r = (n * (n * l * 2.f) - l).normalize();                  // 反射光线
        float spec = pow(std::max(r.z, 0.0f), model->specular(uv));     // 高光
        float diff = std::max(0.f, n * l);                              // 漫反射
        TGAColor c = model->diffuse(uv);                                // 纹理
        //bling-phong模型
        //ambient + diffuse + specular
        //当深度测试通过时shadow为1对整体没有影响 未通过时乘偏移量0.3制造阴影
        for (int i = 0; i < 3; i++) color[i] = std::min<float>(20 + c[i] * shadow * (1.2 * diff + .6 * spec), 255);
        return false;
    }
};
//深度测试
struct DepthShader : public IShader {
    mat<3, 3, float> varying_tri;

    DepthShader() : varying_tri() {}

    virtual Vec4f vertex(int iface, int nthvert) {
        Vec4f gl_Vertex = embed<4>(model->vert(iface, nthvert));            // read the vertex from .obj file
        gl_Vertex = ViewPort * Projection * ModelView * gl_Vertex;          // transform it to screen coordinates
        varying_tri.set_col(nthvert, proj<3>(gl_Vertex / gl_Vertex[3]));    // setting triangle coordinates
        return gl_Vertex;
    }

    virtual bool fragment(Vec3f bar, TGAColor& color) {
        Vec3f p = varying_tri * bar;                        //插值
        color = TGAColor(255, 255, 255) * (p.z / depth);    
        return false;
    }
};

int main(int argc, char** argv) {
    if (2 == argc) {
        model = new Model(argv[1]);
    }
    else {
        model = new Model("obj/african_head.obj");
    }
    //初始化深度数组
    float* zbuffer = new float[width * height];
    shadowbuffer = new float[width * height];
    for (int i = width * height; --i; ) {
        zbuffer[i] = shadowbuffer[i] = -std::numeric_limits<float>::max();
    }

    light_dir.normalize();

    {
        //深度测试渲染
        TGAImage depth(width, height, TGAImage::RGB);
        lookat(light_dir, center, up);
        viewport(width / 8, height / 8, width * 3 / 4, height * 3 / 4);
        projection(0);

        DepthShader depthshader;
        Vec4f screen_coords[3];
        for (int i = 0; i < model->nfaces(); i++) {
            for (int j = 0; j < 3; j++) {
                screen_coords[j] = depthshader.vertex(i, j);
            }
            triangle(screen_coords, depthshader, depth, shadowbuffer);
        }
        depth.flip_vertically(); // to place the origin in the bottom left corner of the image
        depth.write_tga_file("depth.tga");
    }

    Matrix M = ViewPort * Projection * ModelView;

    {
        // rendering
        TGAImage frame(width, height, TGAImage::RGB);
        lookat(eye, center, up);
        viewport(width / 8, height / 8, width * 3 / 4, height * 3 / 4);
        projection(-1.f / (eye - center).norm());
        //uniform_M uniform_MIT uniform_Mshadow
        Shader shader(ModelView, (Projection * ModelView).invert_transpose(), M * (ViewPort * Projection * ModelView).invert());
        Vec4f screen_coords[3];
        for (int i = 0; i < model->nfaces(); i++) {
            for (int j = 0; j < 3; j++) {
                screen_coords[j] = shader.vertex(i, j);
            }
            triangle(screen_coords, shader, frame, zbuffer);
        }
        frame.flip_vertically(); // to place the origin in the bottom left corner of the image
        frame.write_tga_file("framebuffer.tga");
    }

    delete model;
    delete[] zbuffer;
    delete[] shadowbuffer;

    return 0;
}
```

**OpenGL渲染管线**

其中蓝色为可编程部分

![1484423-20181028140845062-1645118386](D:\typora\图片\1484423-20181028140845062-1645118386.png)

1. **顶点数据**从模型中读入的一些三维点的集合。
2. **顶点着色器**对顶点属性做基本处理，求出变换矩阵，进行坐标转换到世界空间。
3. **光栅化**将3D连续的物体转换为离散屏幕像素点，生成供片元着色器使用的片段，即代码中triangle部分。
4. **片元着色器**计算一个像素最终颜色，还包含3D场景的数据（光照、阴影等），在此会进行光照计算和阴影处理等。仅处理单个片元，不能在片元着色器中获取临近片元的信息，以此可保证独立性，实现高并发。



## 一些问题：

### 1.法线贴图：

是一种伪造凹凸光照的技术，用于不使用更多多边形来增添细节，存储形式为RGB图像。RGB分量范围[0,1]，而法线分量范围[-1, 1]，因此需要对每个分量作映射。

在对象空间中（模型顶点的坐标系），法线信息是相对于对象空间朝向，各个方向都有，因此色彩比较丰富。

而在切线空间中（位于三角形表面上的空间）以原法线方向作为z轴正方向，定义一个切线空间，并在切线空间中用三个坐标值就可以表示出偏移量。存储的是**每个点在各自切线空间中的法线偏移**，方向接近[0, 0, 1]因此看上去接近蓝色。

通常使用**切线空间法线纹理**，优势如下：

- 切线空间存储的是相对法线信息，因此换个网格（或者网格变换deforming）应用该纹理，也能得到合理的结果。
- 可以进行uv动画，通过移动该纹理的uv坐标实现凹凸移动的效果
- 可以重用法线纹理，比如,一个砖块,我们仅使用一张法线纹理就可以用到所有的6个面。
- 可以压缩。因为切线空间的法线z方向总是正方向，因此可以仅存储xy方向，从而推到z方向（normal是单位向量，用勾股定理由xy得出z，取z为正的一个即可）。而模型空间的法线纹理方向各异，无法压缩。

**TBN矩阵：**由切线、副切线、法线（世界空间的向量）组成的3x3的矩阵，用以沟通**切向(纹理)空间和世界空间**，上文已说，多使用切线空间法线纹理，而实际需要的是世界空间的法线信息。

​																	 	        **世界空间法线 = 切向空间法线 * TBN矩阵**
$$
\begin{bmatrix}
T_x && T_y && T_z \\
B_x && B_y && B_z \\
N_x && N_Y && N_z
\end{bmatrix}
$$

### 2.各个空间及变换

Vertex Shader输出是在**裁剪空间(Clip Space)**即做完MVP变换后的空间，之后由GPU自己做**透视除法**将顶点转到**标准化设备坐标（NDC）**

![v2-4baca450c51c7f0eea873c42eeda0eef_720w](D:\typora\图片\v2-4baca450c51c7f0eea873c42eeda0eef_720w.png)

而fragment Shader的输入是NDC做**视口变换**后的**屏幕空间（Screen Space）**

过程为：

**(Vertex Shader) => Clip Space => (透视除法) => NDC => (视口变换) => Screen Space => (Fragment Shader)**

项目中有些许不同：GPU自己做透视除法的部分由我们自己实现，即VS后直接得到Screen Space

总过程为：

**模型空间(Model Space) => M矩阵 => 世界空间(World Space) => V矩阵 => 视觉空间(以摄像机为中心)(View Space) => P矩阵 => 裁剪空间 (Clip Space) => Vertex Shader结束 => 透视除法 => 标准化设备空间(NDC) => 视口变换 => 屏幕空间(Screen Space) => 进入Fragmen Shader **

### 3.z-fighting

Z-Fighting 是指当多个不透明的面在世界空间中处于共面时，它们的片元可能在深度缓冲区的值相同(精度误差)，在渲染像素时就会随机绘制，最终导致在每帧之间出现闪烁的现象。会产生如下效果。

![image-20230808172005668](D:\typora\图片\image-20230808172005668.png)

解决方案：

1. 禁用深度测试。
2. 提升精度，但是会牺牲性能。
3. 拉远近平面，但在近处物体可能被裁剪掉。
4. 坐标偏移，在多个面共面时对面进行偏移分开，但当相机仰角过大时上面的面可能会遮住下面。

