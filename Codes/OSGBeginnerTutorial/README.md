

# OpenSceneGraph 3.0 - Beginner's Guide 源码
## Chapter 03 Creating Your First OSG Program
### 03_MonitorRFC
	- 从 osg::Referenced 派生可以获取引用计数的功能
	- osg::Referenced::referenceCount() ---  可以获取其引用计数
### 03_ReadFromCommandLine
	- 从命令行读取文件
	- osg::ArgumentParser
		- read() 功能
```
	osg::ArgumentParser arguments(&argc, argv);
	std::string filename;
	arguments.read("--model", filename);
```
### 03_RedirectNotifier
	- 设置输出调试信息到文件
	- 派生类 osg::NotifyHandler
		- 重写 notify() 方法
	- osg::setNotifyLevel(): 设置日志输出级别
	- osg::setNotifyHandler(): 设置输出的处理类

## Chapter 04 Building Geometry Models
### 04_ColoredQuad
	- 绘制一个四边形
	- Geode
		- Geometry
			- VertexArray
			- NormalArray
			- ColorArray
			- PrimitiveSet
```
	// 创建一个四边形的 geode
	osg::ref_ptr<osg::Geometry> geom = new osg::Geometry;
	osg::ref_ptr<osg::Vec3Array> verts = new osg::Vec3Array;
	verts->push_back(osg::Vec3(0.0f, 0.0f, 0.0f));
	verts->push_back(osg::Vec3(1.0f, 0.0f, 0.0f));
	verts->push_back(osg::Vec3(1.0f, 0.0f, 1.0f));
	verts->push_back(osg::Vec3(0.0f, 0.0f, 1.0f));
	osg::ref_ptr<osg::Vec3Array> normals = new osg::Vec3Array;
	normals->push_back(osg::Vec3(0.0f, -1.0f, 0.0f));
	osg::ref_ptr<osg::Vec3Array> colors = new osg::Vec3Array;
	colors->push_back(osg::Vec3(1.0f, 0.0f, 0.0f));
	colors->push_back(osg::Vec3(0.0f, 1.0f, 0.0f));
	colors->push_back(osg::Vec3(0.0f, 0.0f, 1.0f));
	colors->push_back(osg::Vec3(1.0f, 1.0f, 1.0f));
	geom->setVertexArray(verts.get());
	geom->setNormalArray(normals.get());
	geom->setColorArray(colors.get());
	geom->setNormalBinding(osg::Geometry::BIND_OVERALL);
	geom->setColorBinding(osg::Geometry::BIND_PER_VERTEX);
	geom->addPrimitiveSet(new osg::DrawArrays(GL_QUADS, 0, 4));
	osg::ref_ptr<osg::Geode> geode = new osg::Geode;
	geode->addDrawable(geom.get());
```
### 04_DrawOctahedron
	- 绘制一个多面体, 主要是讲图元, osg::DrawElementsUInt 可以创建一个索引数组, geometry 调用 addPrimitiveSet() 添加它.
	- osgUtil::SmoothingVisitor::smooth 平滑一个几何体, 主要是可以为一个几何体生成法线
```
	osg::ref_ptr<osg::Geometry> geom = new osg::Geometry;
	geom->setVertexArray(vertices.get());
	geom->addPrimitiveSet(indices.get());
	osgUtil::SmoothingVisitor::smooth(*geom);

	osg::ref_ptr<osg::Geode> geode = new osg::Geode;
	geode->addDrawable(geom.get());
```
### 04_OpenGLTeapot
	- 自定义 drawable, 进行自定义绘制, 从 osg::Drawable 派生， 重写下面的两个函数即可实现
	- osg::Drawable::computeBound() --- 返回对应的围绕盒
	- osg::Drawable::drawImplementation() --- 绘制部分， 其内可以调用opengl的相关函数进行绘制。
### 04_PrimitiveFunctor
	- 仿函数, 可以统计多边形面信息, 写一个 operator 用于其他的仿函数继承
	- 本例介绍了 osg::TriangleFunctor, 这里使用其打印面信息，调用模板类的 operator() 函数进行自定义操作, osg::TriangleFunctor 在模拟绘制三角形时调用.
	- 主要原理 osg::Drawable::accept(Functor), 使用 Functor 的 operator() 从模板类中继承
### 04_SimpleObject
	- 几个简单的形状 box, sphere, cone
	- osg::ShapeDrawable::setShape()
	- geode 可以添加多个几何体，几何体也为 Drawable 的派生类
```
	osg::ref_ptr<osg::ShapeDrawable> shape1 = new osg::ShapeDrawable;
	osg::ref_ptr<osg::ShapeDrawable> shape2 = new osg::ShapeDrawable;
	osg::ref_ptr<osg::ShapeDrawable> shape3 = new osg::ShapeDrawable;
	shape1->setShape(new osg::Box(osg::Vec3(-3.0f, 0.0f, 0.0f), 2.0f, 2.0f, 1.0f));
	shape2->setShape(new osg::Sphere(osg::Vec3(3.0f, 0.0f, 0.0f), 1.0f));
	shape3->setShape(new osg::Cone(osg::Vec3(0.0f, 0.0f, 0.0f), 1.0f, 1.0f));
	shape2->setColor(osg::Vec4(0.0f, 0.0f, 1.0f, 1.0f));
	shape3->setColor(osg::Vec4(0.0f, 1.0f, 0.0f, 1.0f));

	osg::ref_ptr<osg::Geode> geode = new osg::Geode;
	geode->addDrawable(shape1.get());
	geode->addDrawable(shape2.get());
	geode->addDrawable(shape3.get());
```
### 04_TessellatePolygon
	- 凹多边形转凸多边形(分形化一个 Geometry)
	- osgUtil::Tessellator: 分形化一个包含多边形边的几何体
		- retessellatePolygons() 分形化一个几何体
### 05_AddModel
	- osg::Group::addChild

## Chapter 05 Managing Scene Graph
### 05_AnalyzeStructure
	- 遍历节点, 打印出其内部的库名和类名
	- 从 osg::NodeVisitor 派生一个 Visitor
		- 构造函数设置遍历模式 setTraversalMode(), 本例使用 osg::NodeVisitor::TRAVERSE_ALL_CHILDREN 遍历所有子节点
		- 重写 apply 函数, 分别处理 osg::Node& node 和 osg::Geode& geode, 注意函数最后要调用 traverse() 方法
		- osg::Object::libraryName() 输出库名, osg::Object::className() 输出类名
		- osg::Geode::getNumDrawables() 获取 Geode 所有可绘制对象的数量, osg::Geode::getDrawable() 获取某个可绘制对象.
### 05_LodNode
	- osgUtil::Simplifier --- 简化一个节点
```
osgUtil::Simplifier simplifier;
simplifier.setSampleRatio(0.5);
model2->accept(simplifier);
```
	- osg::LOD --- addChild, 添加子节点, 根据距离设置节点
### 05_ProxyNode
	- osg::ProxyNode
		- setFileName --- 显示的时候再加载
### 05_SwitchAnimate
	- osg::Switch
		- setValue 设置可见性
		- 重写 osg::Switch::traverse() 方法, 其在各种遍历会调用, 如更新遍历, 剔除遍历
	- 本例通过遍历次数实现自动切换模型
### 05_SwitchNode
	- osg::Switch
		- addChild 方法, 添加节点时设置其可见状态
### 05_TranslateNode
	- osg::MatrixTransform 矩阵变换节点
		- setMatrix
	- osg::Matrix::translate

## Creating Realistic Rendering Effects
### 06_BezierCurve
	- 使用几何着色器(Geometry)创建贝塞尔曲线， 几何着色器插值生成顶点
	- 设置图元类型为 GL_LINE_STRIP， 其在几何着色器的输入类型为 GL_LINES_ADJACENCY_EXT
		- 通过 osg::Program::setParameter 设置参数, 本例三个参数 GL_GEOMETRY_VERTICES_OUT_EXT 和 GL_GEOMETRY_INPUT_TYPE_EXT, GL_GEOMETRY_OUTPUT_TYPE_EXT
	- 使用 line adjacency 图元作为输入, 该输入有四个元素
	- osg::LineWidth 设置线宽
	- 实现如下
```
// 1. 创建几何体， 四个控制点构成 LINES ADJACENCY 图元
	osg::ref_ptr<osg::Vec3Array> vertices = new osg::Vec3Array;
	vertices->push_back(osg::Vec3(0.0f, 0.0f, 0.0f));
	vertices->push_back(osg::Vec3(1.0f, 1.0f, 1.0f));
	vertices->push_back(osg::Vec3(2.0f, 1.0f, -1.0f));
	vertices->push_back(osg::Vec3(3.0f, 0.0f, 0.0f));
	osg::ref_ptr<osg::Geometry> controlPoints = new osg::Geometry;
	controlPoints->setVertexArray(vertices.get());
	controlPoints->addPrimitiveSet(
		new osg::DrawArrays(GL_LINES_ADJACENCY_EXT, 0, 4));
	osg::ref_ptr<osg::Geode> geode = new osg::Geode;
	geode->addDrawable(controlPoints.get());

// 2. 顶点着色器仅仅是传递每个顶点的位置
	#version 120
	#extension GL_EXT_geometry_shader4 : enable
	void main()
	{ gl_Position = ftransform(); }

// 3. 几何着色器, 发送 segments 个顶点， 根据四个控制点生成 segment 个顶点的位置。
	#version 120
	#extension GL_EXT_geometry_shader4 : enable
	uniform int segments;
	void main(void)
	{
	    float delta = 1.0 / float(segments);
	    vec4 v;
	    for ( int i=0; i<=segments; ++i )
	    {
	        float t = delta * float(i);
	        float t2 = t * t;
	        float one_minus_t = 1.0 - t;
	        float one_minus_t2 = one_minus_t * one_minus_t;
	        v = gl_PositionIn[0] * one_minus_t2 * one_minus_t + 
	            gl_PositionIn[1] * 3.0 * t * one_minus_t2 +
	            gl_PositionIn[2] * 3.0 * t2 * one_minus_t +
	            gl_PositionIn[3] * t2 * t;
	        gl_Position = v;
	        EmitVertex();
	    }
	    EndPrimitive();
	}

// 4. 设置着色器和几何着色器的输入输出类型和顶点输出的数量
	int segments = 10;
	osg::ref_ptr<osg::Program> program = new osg::Program;
	program->addShader(
		new osg::Shader(osg::Shader::VERTEX, vertSource));
	program->addShader(
		new osg::Shader(osg::Shader::GEOMETRY, geomSource));
	program->setParameter(GL_GEOMETRY_VERTICES_OUT_EXT, segments + 1);
	program->setParameter(GL_GEOMETRY_INPUT_TYPE_EXT,
		GL_LINES_ADJACENCY_EXT);
	program->setParameter(GL_GEOMETRY_OUTPUT_TYPE_EXT,
		GL_LINE_STRIP);
```
### 06_CartoonCow
	- 使用着色器进行卡通着色
	- 通过光照方向和法线的点乘结果确定颜色
### 06_Fog
	- osg::Fog 的使用
		- setMode，setStart，setEnd，setColor
### 06_Lighting
	- osg::Light 从 osg::StateAttribute 派生
		- setLightNum() --- 设置用的是 OpenGL 的哪个光源
		- setDiffuse / setPosition
	- 创建 osg::LightSource --- 光源实体, 从 osg::Group 派生而来, 定义场景中的一个光源
		- 创建 osg::Light --- 光源属性
		- setLight
	- 用 osg::MatrixTransform 将 LightSource 添加至场景中
```
// 光照打开和关掉
	root->getOrCreateStateSet()->setMode(GL_LIGHT0,
		osg::StateAttribute::ON);
	root->getOrCreateStateSet()->setMode(GL_LIGHT1,
		osg::StateAttribute::ON);
```
### 06_PolygonMode
	- 线框模式绘制
	- osg::PolygonMode
		- setMode
```
	osg::ref_ptr<osg::PolygonMode> pm = new osg::PolygonMode;
	pm->setMode(osg::PolygonMode::FRONT_AND_BACK, osg::PolygonMode::LINE);
	transform1->getOrCreateStateSet()->setAttribute(pm.get());
```
### 06_StateSetInherit
	- 演示状态属性的继承关系
	- osg::StateAttribute::OVERRIDE，
	- osg::StateAttribute::PROTECTED 可以无视上一级的状态
### 06_Texture2D
	- 创建一个四边形，然后贴上纹理
	- osg::Texture2D
		- setImage
### 06_Translucent --- 创建透明物体
	- 创建球体几何体
	- osg::BlendFunc
```
// 设置混合属性
	osg::ref_ptr<osg::BlendFunc> blendFunc = new osg::BlendFunc;
	blendFunc->setFunction(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);

	osg::StateSet* stateset = root->getOrCreateStateSet();
	stateset->setAttributeAndModes(blendFunc);
	stateset->setMode(GL_LIGHTING, osg::StateAttribute::OFF);
	//stateset->setMode(GL_CULL_FACE, osg::StateAttribute::ON);
	stateset->setMode(GL_DEPTH_TEST, osg::StateAttribute::OFF);
	stateset->setRenderingHint(osg::StateSet::TRANSPARENT_BIN);
```
	- 创建一个球体
```
osg::Geode* createSphere(int slices, int stacks, GLfloat radius)
{
	osg::ref_ptr<osg::Vec3Array> vertices = new osg::Vec3Array;
	osg::ref_ptr<osg::Vec3Array> normals = new osg::Vec3Array;
	
	float horz_angle_step = osg::PI * 2.0 / stacks;
	float vert_angle_step = osg::PI / slices;
	float start_vert_angle = -osg::PI_2;
	for (int i = 1; i < slices-1; i++)
	{
		float cur_vert_angle = start_vert_angle + i * vert_angle_step;
		float nxt_vert_angle = start_vert_angle + (i + 1)*vert_angle_step;
		float bottom_height = sin(cur_vert_angle) * radius;
		float top_height = sin(nxt_vert_angle) * radius;
		float bottom_radius = cos(cur_vert_angle) * radius;
		float top_radius = cos(nxt_vert_angle) * radius;
		float bottom_sin = sin(cur_vert_angle);
		float top_sin = sin(nxt_vert_angle);
		float bottom_cos = cos(cur_vert_angle);
		float top_cos = cos(nxt_vert_angle);
		for (int j = 0; j < stacks; j++)
		{
			float cur_horz_angle = j * horz_angle_step;
			float nxt_horz_angle = (j + 1) * horz_angle_step;

			float cur_normal_x = cos(cur_horz_angle);
			float cur_normal_y = sin(cur_horz_angle);
			float nxt_normal_x = cos(nxt_horz_angle);
			float nxt_normal_y = sin(nxt_horz_angle);

			osg::Vec3 left_bottom_vert = osg::Vec3(cur_normal_x*bottom_radius, cur_normal_y*bottom_radius, bottom_height);
			osg::Vec3 right_bottom_vert = osg::Vec3(nxt_normal_x*bottom_radius, nxt_normal_y*bottom_radius, bottom_height);
			osg::Vec3 left_top_vert = osg::Vec3(cur_normal_x*top_radius, cur_normal_y*top_radius, top_height);
			osg::Vec3 right_top_vert = osg::Vec3(nxt_normal_x*top_radius, nxt_normal_y*top_radius, top_height);
			osg::Vec3 left_bottom_normal = osg::Vec3(cur_normal_x*bottom_cos, cur_normal_y*bottom_cos, bottom_sin);
			osg::Vec3 right_bottom_normal = osg::Vec3(nxt_normal_x*bottom_cos, nxt_normal_y*bottom_cos, bottom_sin);
			osg::Vec3 left_top_normal = osg::Vec3(cur_normal_x*top_cos, cur_normal_y*top_cos, top_sin);
			osg::Vec3 right_top_normal = osg::Vec3(nxt_normal_x*top_cos, nxt_normal_y*top_cos, top_sin);

			vertices->push_back(left_bottom_vert);
			vertices->push_back(right_bottom_vert);
			vertices->push_back(right_top_vert);

			vertices->push_back(left_bottom_vert);
			vertices->push_back(right_top_vert);
			vertices->push_back(left_top_vert);

			normals->push_back(left_bottom_normal);
			normals->push_back(right_bottom_normal);
			normals->push_back(right_top_normal);

			normals->push_back(left_bottom_normal);
			normals->push_back(right_top_normal);
			normals->push_back(left_top_normal);
		}

	}

	osg::ref_ptr<osg::Vec4Array> colors = new osg::Vec4Array;
	colors->push_back(osg::Vec4(1.0, 0.0, 0.0, 0.5));

	// normals->push_back(osg::Vec3(0.0f, -1.0f, 0.0f));

	osg::ref_ptr<osg::Geometry> sphere = new osg::Geometry;
	sphere->setVertexArray(vertices.get());
	sphere->setNormalArray(normals.get());
	// sphere->setNormalBinding(osg::Geometry::BIND_OVERALL);
	sphere->setColorArray(colors.get());
	sphere->setColorBinding(osg::Geometry::BIND_OVERALL);
	sphere->addPrimitiveSet(new osg::DrawArrays(GL_TRIANGLES, 0, vertices->size()));
	osg::ref_ptr<osg::Geode> geode = new osg::Geode;
	geode->addDrawable(sphere.get());
	return geode.release();
}
```
	- 从文件中添加着色器
```
	osg::Program* program = new osg::Program;

	osg::ref_ptr<osg::Shader> vertObj = new osg::Shader(osg::Shader::VERTEX);
	osg::ref_ptr<osg::Shader> fragObj = new osg::Shader(osg::Shader::FRAGMENT);
	bool bret = vertObj->loadShaderSourceFromFile(radarVertPath.c_str());
	bret = fragObj->loadShaderSourceFromFile(radarFragPath.c_str());
	program->addShader(vertObj);
	program->addShader(fragObj);
	ss->setAttributeAndModes(program, osg::StateAttribute::ON);
```

## Chapter 07 Viewing the World
### 07_FrameLoop
	- osgViewer::Viewer::getFrameStamp()::getFrameNumber(): 得到帧号
	- 帧循环
```
	while (!viewer.done())
	{
		viewer.frame();
	}
```
### 07_HUD
	- 创建一个相机绘制 HUD 内容，相机有自己的矩阵，而后添加子节点
### 07_MultipleScene(多窗口场景)
	- osgViewer::View --- 创建一个单独的窗口，在其内绘制视图场景, setupViewInWindow 设置窗口位置
	- osgViewer::CompositeViewer::addView --- 添加这样的视图窗口
### 07_MultiSampling(多重采样)
	- osg::DisplaySettings::setNumMultiSamples() --- 设置多重采样
```
osg::DisplaySettings::instance()->setNumMultiSamples(4);
```
### 07_RTT(渲染到纹理)
	- osg::Camera::setRenderTargetImplementation() --- 设置渲染到帧缓存对象
	- osg::Camera::attach() --- 渲染至纹理
	- 从 osg::NodeVisitor 派生一个访问器, 替换其内使用的问题
```
class FindTextureVisitor : public osg::NodeVisitor
{
public:
	FindTextureVisitor(osg::Texture* tex) : _texture(tex)
	{
		setTraversalMode(
			osg::NodeVisitor::TRAVERSE_ALL_CHILDREN);
	}

	virtual void apply(osg::Node& node);
	virtual void apply(osg::Geode& geode);
	void replaceTexture(osg::StateSet* ss)
	{
		if (ss)
		{
			osg::Texture* oldTexture = dynamic_cast<osg::Texture*>(
				ss->getTextureAttribute(0, osg::StateAttribute::TEXTURE)
				);
			if (oldTexture) ss->setTextureAttribute(
				0, _texture.get());
		}
	}

protected:
	osg::ref_ptr<osg::Texture> _texture;
};

void FindTextureVisitor::apply(osg::Node& node)
{
	replaceTexture(node.getStateSet());
	traverse(node);
}
void FindTextureVisitor::apply(osg::Geode& geode)
{
	replaceTexture(geode.getStateSet());
	for (unsigned int i = 0; i < geode.getNumDrawables(); ++i)
	{
		replaceTexture(geode.getDrawable(i)->getStateSet());
	}
	traverse(geode);
}
```
- osg::StateSet 调用 getTextureAttribute 以及osg::StateAttribute::TEXTURE 参数获得纹理.
- 替换所有 Node 和 Geode 的纹理, 如果为 Geode, 则替换其内所包含可渲染对象的所有纹理
- 步骤
	- 创建一个纹理 osg::Texture
	- 创建相机, 设置视口为纹理大小
		- 背景颜色, clearMask, renderOrder
		- setRenderTargetImplementation 为帧缓存
		- attach 一个纹理
		- setReferenceFrame
		- setReferenceFrame( osg::Transform::ABSOLUTE_RF ) 等同于 glLoadMatrix. 要切换回去则用 osg::Transform::RELATIVE_RF
		- setViewMatrixAsLookAt
* 创建帧缓存对象
```
	int tex_width = 1024, tex_height = 1024;
	osg::ref_ptr<osg::Texture2D> texture = new osg::Texture2D;
	texture->setTextureSize(tex_width, tex_height);
	texture->setInternalFormat(GL_RGBA);
	texture->setFilter(osg::Texture2D::MIN_FILTER, osg::Texture2D::LINEAR);
	texture->setFilter(osg::Texture2D::MAG_FILTER, osg::Texture2D::LINEAR);
	osg::ref_ptr<osg::Camera> camera = new osg::Camera;
	camera->setViewport(0, 0, tex_width, tex_height);
	camera->setClearColor(osg::Vec4(0.0f, 0.0f, 0.0f, 0.0f));
	camera->setClearMask(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
	camera->setRenderOrder(osg::Camera::PRE_RENDER);
	camera->setRenderTargetImplementation(osg::Camera::FRAME_BUFFER_OBJECT);
	camera->attach(osg::Camera::COLOR_BUFFER, texture.get());
	camera->setReferenceFrame(osg::Camera::ABSOLUTE_RF);
```

## Chapter 08 Animating Scene Objects
### 08_AnimateCharacter
	- osgAnimation::BasicAnimationManager --- 派生自 osg::NodeCallback
		- getAnimationList() --- 得到动画列表
		- osgAnimation::Animation
    	- playAnimation() --- 应用动画列表中的一个动画
	- 该动画内容保存在 osg 模型内

### 08_AnimationChannel
	- osgAnimation::Vec3Keyframe, osgAnimation::QuatKeyframe 关键帧组成 osgAnimation::Vec3KeyframeContainer 和 osgAnimation::QuatKeyframeContainer 两个容器
	- osgAnimation::Vec3LinearChannel/osgAnimation::QuatSphericalLinearChannel
		- 对于 channel 其有自身的 sampler, 每个sampler 内又包含一个插值器和一个 keyframe 容器.
		- getOrCreateSampler()
			- getOrCreateKeyframeContainer --- 得到容器
			- sampler 的容器插入关键帧.
	- osgAnimation::Animation
		- setPlayMode
		- addChannel
		- 动画播放时根据内包含的频道更新数值.
	- osgAnimation::UpdateMatrixTransform
		- getStackedTransforms()
			- 这是个容器， 保存 osgAnimation::StackedTranslateElement 元素 和 osgAnimation::StackedQuaternionElement 元素，这些名称的名字和之前 channel 的名字一样
		- 这是一个节点更新回调, 之前播放动画的时候, 会更新其内的频道值, 而这些频道值则会更新上面容器内的值.因此更新了该类关联的变换矩阵
	- osgAnimation::BasicAnimationManager
		- registerAnimation 注册一个动画
		- 这也是一个更新回调
		- playAnimation 播放动画
	- 总结如下:
		- 需要设置一个节点的更新回调为 BasicAnimationManger 类型, 主要用途为播放其内的 Animation
		- 矩阵变换节点需要有个更新回调为 UpdateMatrixTransform 类型, 主要用途是使用 Animation 更新后的结果更新矩阵变换节点. 注意该矩阵变换节点使用的矩阵等于频道内的平移和旋转.
		- 动画会包含所有需要更新的频道.
		- 每个频道有自己的sampler, sampler 包含自己的插值器和关键帧容器.
### 08_AnimationPath
	- 实现动画路径
	- osg::AnimationPath
		- setLoopMode: 设置循环模式
		- insert: 插入一个控制点
	- osg::AnimationPathCallback 节点回调
		- setAnimationPath 设置路径(osg::AnimationPath)
	- 通常设置为节点更新回调
### 08_FadingIn
	- 演示了 osg::Matrial 属性以及其相应的回调, 并演示了 osgAnimation::InOutCubicMotion
	- 设置对象的状态属性更新回调
	- osg::StateAttributeCallback 派生自 osg::Callback
		- bool run()
		- void operator(), 两个参数, 分别为 osg::StateAttribute 和 osg::NodeVisitor 指针
			- 本例修改 osg::Material 内 Diffuse 的 alpha 值, osg::Material 从 osg::StateAttribute 指针而来
		- osg::StateAttribute 可以设置更新回调
	- osgAnimation::InOutCubicMotion
		- 本例使用 update 更新, 使用 getValue 获取其值
		- 类型别名 osgAnimation::MathMotionTemplate<InOutCubicFunction>
		- 结构体 MathMotionTemplate 派生自 osgAnimation::Motion
			- osgAnimation::Motion
				- 纯基类函数 getValueInNormalizedRange, 第一个参数是时间比例(当前时间/总时间),第二个参数为返回的结果
			- MathMotionTemplate 重写了函数 getValueInNormalizedRange, 调用了模板类的 getValueAt 函数
		- 结构体 InOutCubicFunction 重写了 getValueAt 函数, 三次方来回运动
### 08_Flashing
	- 设置连续图像动画, 使用 osg::ImageSequence
		- addImage() --- 添加图像 
		- setLength() --- 设置时间长度
		- play() --- 播放
	- 可以学会如何创建 osg::Image
		- allocateImage 分配内存
		- data() 某一行列的数据指针
	- osg::Texture2D 纹理
		- setImage() 目标可以是 osg::ImageSequence
### 08_GeometryDynamically
	- 通过更新回调更改一个几何体, 该回调为 osg::Drawable::UpdateCallback 的派生类
	- 同时演示了 osg::Drawable 的更新回调, 其更新回调重写的是 update() 函数
	- 更新几何数组后调用函数 osg::Geometry::dirtyDisplayList() 和 osg::Geometry::dirtyBound() 更新显示列表和围绕盒
### 08_SwitchUpdate
	- 通过更新回调切换Switch节点子节点的状态

## Chapter 09 Interacting with Outside Elements 
### 09_DrivingCessna
	- osg::Camera
		- setAllowEventFocus() 是否允许其关联窗口产生的事件影响到该相机
		- setViewMatrixAsLookAt() 设置视图矩阵
	- osgViewer::Viewer
		- addEventHandler() 添加事件处理(处理窗口事件)
	- osgGA::GUIEventHandler 派生窗口事件处理类
		- 重写 bool handle() 函数
			- osgGA::GUIEventAdapter 获取键盘鼠标事件
### 09_GCTraits
	- osg::GraphicsContext::Traits 创建一个 traits
	- osg::GraphicsContext::createGraphicsContext 使用该 traits 创建一个窗口 osg::GraphicsContext
	- osg::Camera 创建一个相机
		- setGraphicsContext 设置其窗口上下文
		- setProjectionMatrixAsPerspective 设置投影矩阵
	- osgViewer::Viewer
		- setCamera 设置相机
### 09_PickingGeometry --- 选择框
	- 展示了如何创建选择框
		- 线框,禁光照, 矩阵变换节点
		- 通过矩阵变换绘制和移动线框
	- 鼠标点击创建框框, 通过 MatrixTransform 实现
	- NodeMask 的使用
	- 本例使用相机接受相交访问器(osgUtil::IntersectionVisitor)
	- 鼠标点击相交检测
	- osgUtil::IntersectionVisitor 访问, 使用相机 accept 调用该 visitor
	- osg::computeLocalToWorld 计算矩阵
### 09_UserTimer --- 可以添加一个用户事件
	- 本例是在 Frame 事件中添加用户事件
	- osgGA::GUIEventAdapter::FRAME 可以处理帧事件
	- viewer->getEventQueue()->userEvent() --- 添加一个用户事件
### 09_Win32Handler --- Win32 API 和 osg
	- WM_CREATE 事件创建一个 osgViewer 而后用线程推进
	- 设置 traits->inheritedWindowData = windata(为窗口句柄)


## Chapter 10 Saving and Loading Files
### 10_CustomFormat --- 自定义格式插件读写
	- 派生自 osgDB::ReaderWriter
	- REGISTER_OSGPLUGIN 宏注册

## Chapter 11 Developing Visual Components
### 11_Billboard 公告板
	- osg::Billboard
		- setMode: osg::Billboard::POINT_ROT_EYE
		- addDrawable: 添加一个可绘制对象, 因为 Billboard 派生自 osg::Geode
### 11_Outline 给物体添加轮廓线效果.
	- osgFX::Outline
		- setWidth
		- setColor
		- addChild
### 11_ParticleSystem 粒子系统
	- osgParticle::ParticleSystem
	- osg::PointSprite
	- osgParticle::RandomRateCounter
	- osgParticle::ModularEmitter
	- osgParticle::AccelOperator
	- osgParticle::ModularProgram
### 11_Shadow 阴影效果
	- osg::Node
		- setNodeMask() --- 设置节点的 Mask
	- osg::AnimationPathCallback 动画路径回调
		- setAnimationPath() --- 设置动画路径
	- osg::LightSource 光源
		- getLight()
	- osgShadow::ShadowMap
		- setLight
		- setTextureSize
		- setTextureUnit
	- osgShadow::ShadowedScene 这是一个节点
		- setShadowTechnique 设置阴影技术
		- setReceivesShadowTraversalMask 设置阴影接受 mask
		- setCastsShadowTraversalMask 设置阴影投射 mask
### 11_Text 输出文字
	- osg::Camera 
		- 设置投影矩阵
		- 创建 HUD 相机用于绘制文本
	- osgText::Text
		- setFont
		- setCharacterSize
		- setAxisAlignment
		- setPosition
### 11_Text3D --- 3D 字体, 当成一个 3D 模型
	- osgText::Text3D
		- setFont
		- setCharacterSize
		- setCharacterDepth
		- setAxisAlignment
		- setPosition
		- setText

## Chapter 12 Improving Rendering Efficiency
### 12_MultiThread --- 多线程程序
	- 本例演示键盘输入, 屏幕显示输入内容.
	- OpenThreads::Thread 派生一个线程类
	- OpenThreads::Mutex 互斥信号
	- OpenThreads::ScopedLock<OpenThreads::Mutex> 锁
	- OpenThreads::startThread --- 开启线程
	- osgText::Text
		- setText: 设置文本
### 12_Occluder
	- osg::OccluderNode 派生自 osg::Group, 主要用于提供钩子添加 ConvexPlanarOccluders 至场景中.
		- setOccluder(): 添加一个 ConvexPlanarOccluder
	- osg::ConvexPlanarOccluder 其内有 ConvexPlanarPolygon&
	- osg::ConvexPlanarPolygon
		- add 添加顶点
	- 只需要把 OccluderNode 添加至场景就可以实现遮挡
### 12_QuadTree --- 使用 pageLOD 创建四叉树
	- osg::HeightField 派生自 osg::Shape
		- setSkirtHeight 设置边裙高度
		- setOrigin 设置原点,这里设置左下角为原点
		- setHeight
		- setXInterval, setYInterval
	- osg::PagedLOD
		- insertChild
		- setFileName
		- setCenterMode
		- setCenter
		- setRadius
		- setRange
### 12_SharingTexture
	- 当读写图像文件时, 如果之前已经读取则不用再次访问硬盘, 直接从内存中获取.
	- osgDB::Registry::instance()->setReadFileCallback() 设置读文件回调
	- osgDB::ReadFileCallback 读文件回调类, 重写 readImage 函数
		- osgDB::Registry::instance()->readImageImplementation 调用底层的实现
	- osgDB::Registry::instance()->getOrCreateSharedStateManager 创建共享状态管理器
	- osgDB::SharedStateManager
		- share
### 12_ThreadingModel
	- osgViewer::ViewerBase::ThreadingModel 线程模型
		- osgViewer::ViewerBase::AutomaticSelection
		- osgViewer::ViewerBase::SingleThreaded
		- osgViewer::ViewerBase::ThreadPerContext
		- osgViewer::ViewerBase::ThreadPerCamera
	- osgViewer::CompositeViewer
		- addView 添加多个 View
		- setThreadingMode --- 设置线程模式
