## blender shading 界面、材质、渲染

### shading 界面

在最顶部的菜单栏，点击 shading 就可以切换到 shading 界面。

![./shading.png](shading.png)

这里为了下文描述方便，给窗口中各个区域定义一个名字。

把最顶部称为菜单栏。

中间这一大块显示模型的部分称为模型窗口。模型窗口分为顶部的菜单栏，左上角的按钮，右上角的按钮。模型窗口右上角的按钮的右边还有个小三角标志，点开后展示的那个称为插件栏。按`n`打开或关闭模型窗口的插件栏。

右边这一部分，分为上下两块，上面称为场景管理窗口，下面称为属性窗口。

#### 模型显示模式

模型窗口，菜单栏最右边，四个球的图标。

当切换到 shading 界面时，会自动切换到第3个（display in material preview mode）材质模式。

第4个（display render preview）是渲染模式，如果场景中有灯光的话，切换到这个模式应该就能看见影子了。

#### 材质

选择一个模型。

属性窗口，左侧选择第14个按钮（material properties），当选中的模型没有材质时，这时右侧是空的。

点击 new 就会给这个选中的模型创建材质，这时右侧就会显示属性列表了。

如果多个模型想用一样的材质，可以点击 new 左边那个按钮（browse material to be linked），可以选择已经创建的材质。

#### 渲染

##### 渲染引擎

属性窗口，左侧选择第2个按钮（render properties），右侧最上面 render engine 可以选择渲染引擎。

默认是 eevee 引擎，场景是实时渲染的，有很多效果是在欺骗人的眼睛。

cycles 引擎有光追，光的反射、折射效果更好，渲染出的效果更加真实。

eevee 菜单下面勾选 ambient occlusion 后，可以控制两个物体接近的位置上的阴影效果。

eevee 菜单下面勾选 screen space reflections 后，可以控制渲染出倒影的效果。

##### 阴影

切换到渲染模式后，阴影有明显的像素感。

属性窗口，左侧选择第2个按钮（render properties），右侧点击 shadows 标签，然后勾选 soft shadows。这样阴影就会有平滑的效果。

##### 渲染

最顶上菜单栏，render->render image，就会在新窗口开始渲染图片了。

渲染完成的图片，在新窗口最顶部菜单栏，image->save，就可以保存了。

