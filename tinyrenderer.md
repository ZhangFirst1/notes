## tinyrenderer

​	跟着[ssloy/tinyrenderer: A brief computer graphics / rendering course (github.com)](https://github.com/ssloy/tinyrenderer)项目实现一个基础的渲染器。

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
3. games101中，实现Projection变换是先获得视锥（也就是项目中投影变换的效果），然后把视锥远裁剪面缩小到与近裁剪面相同大小，再按照正交投影将包裹所有可见模型物体的虚拟长方体转换到取值[-1,1] [-1,1]的2 * 2 * 2的立方体中（长方体转为立方体是为了方便后续的裁剪以及其他操作的计算）。所以二者透视矩阵并不相同。
4. games101中，近平面是自定义的，且投影到近平面上，而项目中默认是投影到xoy面（计算时不区分近远平面）。
5. 视口变换与games101中相同。
