# creo二次开发

### 1，版本信息

cero 4.0 M060

### 2，安装教程

链接：https://pan.baidu.com/s/1ZWnReB2Fl3mDLyn_cU_29Q 
提取码：8t6w 

### 3，生成python版的vb API

由于Creo没有提供专门用于Python的开发工具包，所以只能考虑借用现有的开发工具包。VBAPI实际是对Creo二次开发函数的COM封装，所以一般Windows下可以调用COM组件的语言其实都可以利用VBAPI进行Creo的二次开发。Python可以使用一个第三方库win32com操作COM对象，故Python可以利用VBAPI二次开发工具包进行二次开发。

1, VBAPI环境配置

```
添加PRO_COMM_MSG_EXE到环境变量

值为: D:\PRTC\PTC\Creo 4.0\M060\Common Files\x86e_win64\obj\pro_comm_msg.exe
```

2, 注册COM服务器

```
以管理员权限运行Creo安装目录下子目录"Parametric/bin"中的vb_api_register.bat文件即可
```

3, 安装pywin32

```
pip3 install pywin32 -i http://pypi.douban.com/simple/ --trusted-host=pypi.douban.com/simple
```

4, 配置pywin32

```
cd C:\Users\wgl\PycharmProjects\testlocust\venv\Lib\site-packages\win32com\client

python makepy.py
```

弹框如下：

![image](https://github.com/Mountains-and-rivers/creo-second-dev/blob/main/images/coreAPI.png)

选择 Creo VB API Type Library For Creo Parametric 4.0 {1.0}

该命令会在 C:\Users\wgl\AppData\Local\Temp\gen_py\3.9目录下生成文件:

```
176453F2-6934-4304-8C9D-126D98C1700Ex0x1x0.py  
```

这就是转换后的python版的vb api

改名为VBAPI.py  方便python 调用。

###  4，关键开发技术

```
4.1 关键类的处理

VB API采用面向对象的方法对CREO操作进行了封装，在编写程序过程中只需调用这些类即可。VB API帮助文档中指出，这些类的主要类型[10]包括：

1) Creo Parametric-Related Classes。形似IpfcXXX的类。这些类不能用New关键词进行初始化，只能通过程序中已创建或列出对象的方法获得对应的句柄进行赋值初始化。

2) Module-Level Classes。形似CMpfcXXX的类。包含静态方法用于初始化某些VB对象。

3) Compact Data Classes。形似CCpfcXXX的类。这些类只用于存储数据。主要用于存储和处理VB API中方法的返回数据。

4) Enumeration Classes。枚举类。

此外，还有Array Classes、Sequence Classes等数据结构类用于存储相关数据。

Creo Parametric-Related Classes和Module-Level Classes类用于初始化相关选项、获得Creo的相关数据以及获得Creo Parametric-Related Classes对象，在Python中使用win32com.client.Dispatch(uuid)方法生成。Dispatch的参数uuid可以在前文所述使用makepy.py生成的文件中查找得到。

Creo Parametric-Related Classes类似C语言的指针的概念，对其操作相当于直接操作CREO的内存数据，只能通过Creo Parametric-Related Classes或Module-Level Classes的方法或属性获得。

Enumeration Classes为枚举类型，其值也可以makepy.py生成的文件中查找得到。

4.2 类的继承

Python为动态类型的语言，子类调用父类的属性方法无须进行类型转换，直接调用即可。此外，Python可以自动实现VB API中的多次类型转换。例如IpfcSolid的父类分别为IpfcModel和IpfcFamliyTableRow，当系统获得一个IpfcModel对象时，如果能够确定也是Ipfcsolid对象，则该对象可以直接调用IpfcFamliyTableRow类的属性和方法，无须像VB那样经过多次显式类转换。

4.3 应用实例

以批量添加和清空零件关系为例，对本文介绍的方法进行验证。Python为3.7，Creo版本为2.0。首先按照前文配置好环境。根据官方文档，启动Creo会话只需调用CCpfcAsyncConnection.Start方法即可生成Creo会话对象。在win32com生成的文件中查找CCpfcAsyncConnection的uuid为{456E0110-2031-3907-AFE5-9201C97A915E}，故启动Creo会话关键代码如下：

cAC=client.Dispatch('{456E0110-2031-3907-AFE5-9201C97A915E}')

AsyncConnection = cAC.Start(creoapp， '') #creoapp为creo路径

启动会话后，需要枚举目录包含的零件，关键代码如下：

files = AsyncConnection.Session.ListFiles("*.prt"， EpfcFILE_LIST_LATEST， INPUT_DIR)

修改关系需要将文件加载到内存中。CCpfcModelDescriptor和CCpfcRetrieveModelOptions类主要用于生成打开文件的选项，利用Creo会话对象的RetrieveModelWithOpts调用这两个对象即可实现将零件加载到内存中，关键代码如下：

ModelDescriptor = client.Dispatch(‘{74D4E90E-031B-3734-8CE1-36D5730A6728})

descmodel=ModelDescriptor.Create(1， ''， None)

descmodel.Path=files.Item(i)#files.Item(i)为要导出文件路径

RetrieveModelOptions=client.Dispatch('{2264B49E-C652-384F-AB53-71B57DA275BE}')

options=RetrieveModelOptions.Create()

options.AskUserAboutReps = False

model=AsyncConnection.Session.RetrieveModelWithOpts(descmodel， options)

加载到内存的model为IpfcModel对象，如前文所述，Python可以无须类型转换，对象直接调用IpfcRelationOwner类的方法即可完成相关关系的操作。添加关系代码如下：

originrels = model.Relations

for j in range(0， originrels.Count)：

relations.Append(originrels.Item(j))

for line in rel_contents：

relations.Append(line)

model.Relations = relations

删除关系代码如下：

model.DeleteRelations()
```

### 5,python 二次开发creo demo

##### 5.1 pyCharm 设置

由于VBAPI.py 文件较大，因此要修改以下参数为99999

![image](https://github.com/Mountains-and-rivers/creo-second-dev/blob/main/images/vmSet.png)

5.2 python 调用 vb api

```
# -*- coding: utf8 -*-
import win32com
from win32com import client
import VBAPI
import tkinter
from tkinter import scrolledtext, messagebox, filedialog, Tk, Button, Entry, Label
import os

CREO_APP = 'C:/PTC/Creo 2.0/Parametric/bin/parametric.exe'
PART_DIR = 'D:/mydoc/creo_python/fin.prt'
OUTPUT_DIR = 'D:/test/'

win = Tk()
win.title("批量将文件的族表对象导出到文件")
win.resizable(0, 0)

Label(win, text="Creo程序路径").grid(row=0, column=0, sticky='W')
Label(win, text="要导出的文件").grid(row=1, column=0, sticky='W')
Label(win, text="导出目录").grid(row=2, column=0, sticky='W')

e1 = Entry(win, width="45")
e2 = Entry(win, width="45")
e3 = Entry(win, width="45")
e1.grid(row=0, column=1, padx=5, pady=5)
e2.grid(row=1, column=1, padx=5, pady=5)
e3.grid(row=2, column=1, padx=5, pady=5)
e1.insert(0, CREO_APP)
e2.insert(0, PART_DIR)
e3.insert(0, OUTPUT_DIR)


def convert():
    cAC = client.Dispatch(VBAPI.CCpfcAsyncConnection)
    AsyncConnection = cAC.Start(CREO_APP + ' -g:no_graphics -i:rpc_input', '')
    ModelDescriptor = client.Dispatch(VBAPI.CCpfcModelDescriptor)
    descmodel = ModelDescriptor.Create(getattr(VBAPI.constants, "EpfcMDL_PART"), "", None)
    descmodel.Path = PART_DIR
    RetrieveModelOptions = client.Dispatch(VBAPI.CCpfcRetrieveModelOptions)
    options = RetrieveModelOptions.Create()
    options.AskUserAboutReps = False
    model = AsyncConnection.Session.RetrieveModelWithOpts(descmodel, options)
    AsyncConnection.Session.ChangeDirectory(OUTPUT_DIR)
    familyTableRows = model.ListRows()
    for i in range(0, familyTableRows.Count):
        familyTableRow = familyTableRows.Item(i)
        instmodel = familyTableRow.CreateInstance()
        instmodel.Copy("m_" + instmodel.InstanceName + ".prt", None)
    AsyncConnection.End()
    tkinter.messagebox.showinfo('提示', '文件已导出完毕')
    os.startfile(OUTPUT_DIR)


def chooseapp():
    filename = tkinter.filedialog.askopenfilename()
    if filename != '':
        CREO_APP = filename
        e1.delete('0', 'end')
        e1.insert(0, CREO_APP)


def choosepart():
    filename = tkinter.filedialog.askopenfilename()
    if filename != '':
        PART_DIR = filename
        e2.delete('0', 'end')
        e2.insert(0, PART_DIR)


def choosedir():
    dirname = tkinter.filedialog.askdirectory()
    if dirname != '':
        OUTPUT_DIR = dirname
        e3.delete('0', 'end')
        e3.insert(0, OUTPUT_DIR)


Button(win, text="选择文件", command=chooseapp).grid(row=0, column=2, padx=5, pady=5)
Button(win, text="选择文件", command=choosepart).grid(row=1, column=2, padx=5, pady=5)
Button(win, text="选择路径", command=choosedir).grid(row=2, column=2, padx=5, pady=5)
Button(win, text="导出", command=convert).grid(row=3, column=0, sticky='W', padx=5, pady=5)
Button(win, text="退出", command=win.quit).grid(row=3, column=2, sticky='E', padx=5, pady=5)

win.mainloop()
```

效果如下：

![image](https://github.com/Mountains-and-rivers/creo-second-dev/blob/main/images/pythonDemo.png)

##### 5.1 转换前后API对比

vb API

![image](https://github.com/Mountains-and-rivers/creo-second-dev/blob/main/images/drawing.png)

注意看IpfcBaseSession和CreateDrawingFromTemplate在VBAPI.py中的位置

![image](https://github.com/Mountains-and-rivers/creo-second-dev/blob/main/images/v1.png)

![image](https://github.com/Mountains-and-rivers/creo-second-dev/blob/main/images/v2.png)

![image](https://github.com/Mountains-and-rivers/creo-second-dev/blob/main/images/v3.png)

![image](https://github.com/Mountains-and-rivers/creo-second-dev/blob/main/images/v4.png)

uuid 和函数的对应关系搞清楚 就可以找到对应的vb api 接口  进行绘图了。结合第4节 关键技术理解 待探究

### 6，自动绘图 vb api （2.0 M060 版本）

Creo二次开发自动出图一直是热烈讨论的话题。出图的工作是其实也是设计的工作，特别是尺寸、公差等标注更是需要工程人员大量的知识、经验积累才能完成。通过二次开发可能在特定场合能够完成自动出图的工作，想做一个通用的全自动出图至少目前是很难做到，不过可以通过二次开发做一些预置的辅助工作，减少设计人员的一些机械化常规工作。

##### 6.0 查看creo 下的vb api

路径：

```
D:\PRTC\PTC\Creo 4.0\M060\Common Files\vbapi\vbapidoc
```

![image](https://github.com/Mountains-and-rivers/creo-second-dev/blob/main/images/vbapi.png)

##### 6.1 同名绘图的创建

生成绘图文件可以使用ProDrawingFromTmpltCreate函数完成，其中第一个参数为文件名，第二个参数为绘图模板文件，只需如下调用即可

```
status = ProDrawingFromTmpltCreate(data.name, wtemplatename, &model, options, &created_drawing, &errors);
```

其中name可以通过获取当前模型的模型名获得：

```
status = ProMdlDataGet(mdl, &data);
```

##### 6.2 视图的基本操作

视图在Toolkit中使用ProView句柄进行描述，对视图的修改最常见的包括以下几种：

##### 6.2.1 视图的位置和比例

如果视图的ProView句柄已获得，则可以通过`ProDrawingViewMove`和`ProDrawingViewScaleSet`设定其位置和比例，注意移动视图的坐标系采用的是Screen coordinate system，而这两个操作都必须保证视图在当前窗口打开。对于希望以程序生成的视图，其位置一般都可以通过生成对应视图的函数参数设定，留在第三节进行说明。

##### 6.2.2  视图的样式

视图的样式由proDrawingViewDisplay这个结构体进行描述:

```
typedef struct proDrawingViewDisplay
{
  ProDisplayStyle style;
  ProBoolean quilt_hlr;
  ProTanedgeDisplay tangent_edge_display;
  ProCableDisplay cable_display;
  ProBoolean concept_model;
  ProBoolean weld_xsec;
} ProDrawingViewDisplay;
```

结构体中style表述视图的显示方式，使用了一个enum数据进行描述：

```
typedef enum hlr_disp
{
   PRO_DISPSTYLE_DEFAULT = 0,
   PRO_DISPSTYLE_WIREFRAME,
   PRO_DISPSTYLE_HIDDEN_LINE,
   PRO_DISPSTYLE_NO_HIDDEN,
   PRO_DISPSTYLE_SHADED,
   PRO_DISPSTYLE_FOLLOW_ENVIRONMENT,
   PRO_DISPSTYLE_SHADED_WITH_EDGES
} ProDisplayStyle;
```

ProDrawingViewDisplay结构体数据相对复杂，在实际代码撰写过程中，可以通过先获取当期视图样式，再修改对应值的方式减少工作量。获取和设定视图样式分别由`ProDrawingViewDisplayGet`h和`ProDrawingViewDisplaySet`两个函数完成，所以设定视图的显示方式代码如下：

```
ProError _setDisplayStyle(ProDrawing drawing, ProView view, ProDisplayStyle style)
{
  ProError status;
  ProDrawingViewDisplay displayStatus;
  status = ProDrawingViewDisplayGet(drawing, view, &displayStatus);
  displayStatus.style = style;
  status = ProDrawingViewDisplaySet(drawing, view, &displayStatus);
  return status;
}
```

##### 6.3 视图的创建

视图其实可以根据绘图模板文件直接生成，不过存在一定的局限性，例如同一类型的零件的尺寸、宽高等特征在生成视图时可能需要设置不同的比例和方向，如果存在这些情况则根据绘图模板文件直接生成则存在一定的困难，所以下面介绍视图的创建。

##### 6.3.1 主视图的创建

在Creo中，主视图是投影视图的基础，确定主视图后俯视图以及左视图只需要通过投影的方式即可完成。在完成主视图的创建是，首先需要明确主视图的摆放方向。通常情况下一类零件的主视图方向是一定的，但存在宽高等特征的问题导致主视图可能需要旋转确定。确定零件或装配体得外形尺寸可参照[CREO Toolkit二次开发-外形尺寸](https://www.hudi.site/2020/12/01/CREO Toolkit二次开发-外形尺寸/)一文。而零件的旋转后得到视图的位置则可以通过位姿矩阵确定，详见[CREO 二次开发—位姿矩阵详解](https://www.hudi.site/2021/09/14/CREO二次开发-位姿矩阵介绍/)一文。创建主视图使用`ProDrawingGeneralviewCreate`函数，示例代码如下：

```
status = ProDrawingCurrentsolidGet(drawing, &solid);
status = ProDrawingCurrentSheetGet(drawing, &sheet);

//////////////定义摆放点，使用Screen coordinate system
refPoint[0] = 200;
refPoint[1] = 600;
refPoint[2] = 0;

//////////////定义摆放方向,FRONT，设置比例0.015，显示方式为PRO_DISPSTYLE_HIDDEN_LINE
for (int i = 0; i < 4; i++)
{
  for (int j = 0; j < 4; j++)
  {
    matrix[i][j] = i == j ? 1 : 0;
  }
}
status = ProDrawingGeneralviewCreate(drawing, solid, sheet, PRO_B_FALSE, refPoint, 1, matrix, &positive_view);
status = _setDisplayStyle(drawing, positive_view, PRO_DISPSTYLE_HIDDEN_LINE);
status = ProDrawingViewScaleSet(drawing, positive_view, 0.15);
```

##### 6.3.2 投影视图的创建

投影视图可以根据给定的视图以及新视图的位置确定，此视图比例与给定的视图一致，无法修改，通过`ProDrawingProjectedviewCreate`函数生成。接上面生成的主视图为例，生成俯视图只需把位置定在主视图的下方然后即可生成：

```
refPoint[1] -= 200;
status = ProDrawingProjectedviewCreate(drawing, positive_view, PRO_B_FALSE, refPoint, &top_view);
status = _setDisplayStyle(drawing, top_view, PRO_DISPSTYLE_HIDDEN_LINE);
```

创建三视图效果如下图所示：

![image](https://github.com/Mountains-and-rivers/creo-second-dev/blob/main/images/1.gif)

##### 6.3.3 剖视图的创建

剖视图的创建和投影视图一样，使用`ProDrawingView2DSectionSet`函数设定其截面即可：

```
status = ProDrawingProjectedviewCreate(drawing, parentView, PRO_B_FALSE, refPoint, &_2DSectionView);
status = ProDrawingView2DSectionSet(drawing, _2DSectionView, L"TESTSEC", PRO_VIEW_SECTION_AREA_FULL, NULL, NULL, parentView);
```

创建剖视图效果如下图所示：

![image](https://github.com/Mountains-and-rivers/creo-second-dev/blob/main/images/2.gif)

##### 6.3.4 辅助视图的创建

辅助视图的创建由`ProDrawingViewAuxiliaryCreate`函数完成，指定对应的投影边和办法位置即可，相对简单，直接给出代码：

```
status = ProSelect((char *)"edge", 1, NULL, NULL, NULL, NULL, &sel, &n_sel);
if (status == PRO_TK_NO_ERROR)
{
  status = ProDrawingViewAuxiliaryCreate(drawing, *sel, point, &auxiliaryView);
  status = _setDisplayStyle(drawing, auxiliaryView, PRO_DISPSTYLE_HIDDEN_LINE);
  status = ProDwgSheetRegenerate(drawing, sheet);
}
```

创建辅助视图效果如下图所示：

![image](https://github.com/Mountains-and-rivers/creo-second-dev/blob/main/images/3.gif)

##### 6.3.5 辅助视图的创建

创建详细视图通过`ProDrawingViewDetailCreate`函数实现，具体操作与投影视图和辅助视图类似，只是函数参数的样条曲线的创建相对复杂一点，样条曲线的在Toolkit的数据结构描述如下：

```
typedef struct ptc_spline
{
  int         type;
  double     *par_arr;        /* ProArray of spline parameters */
  ProPoint3d *pnt_arr;        /* ProArray of spline interpolant points */
  ProPoint3d *tan_arr;        /* ProArray of tangent vectors at each point */
  int         num_points;     /* Size for all the arrays */
} ProSplinedata;
```

利用`ProSplinedataInit`可以根据给定样条曲线参数、样条线插值点和各点的切线向量生成目标样条曲线，直接从Toolkit的示例代码中找到创建样条曲线的代码并修改为选择边上下左右格偏移20的四个点，示例代码如下：

```
//下面两个函数直接拷贝官方帮助文件
/*====================================================================*\
    FUNCTION :    ProUtilVectorDiff()
    PURPOSE  :    Difference of two vectors
\*====================================================================*/
double *ProUtilVectorDiff(double a[3], double b[3], double c[3])
{
  c[0] = a[0] - b[0];
  c[1] = a[1] - b[1];
  c[2] = a[2] - b[2];
  return (c);
}

/*====================================================================*\
    FUNCTION :    ProUtilVectorLength()
    PURPOSE  :    Length of a vector
\*====================================================================*/
double ProUtilVectorLength(double v[3])
{
  return (sqrt(v[0] * v[0] + v[1] * v[1] + v[2] * v[2]));
}

ProError _coordsolidtoScreen(ProView view, ProPoint3d pointsolidCoord, ProPoint3d pointScreenCoord)
{
  ProError status;
  ProMdl mdl;
  ProSolid solid;

  ProMatrix transSolidtoScreen;
  status = ProMdlCurrentGet(&mdl);
  status = ProDrawingCurrentsolidGet(ProDrawing(mdl), &solid);
  status = ProViewMatrixGet(ProMdl(solid), view, transSolidtoScreen);
  status = ProPntTrfEval(pointsolidCoord, transSolidtoScreen, pointScreenCoord);
  return status;
}
drawing = (ProDrawing)mdl;
status = ProDrawingCurrentSheetGet(drawing, &sheet);
status = ProDrawingCurrentsolidGet(drawing, &solid);

AfxMessageBox(_T("请选择一个边以生成详细视图。"));
status = ProSelect((char *)"edge", 1, NULL, NULL, NULL, NULL, &sel, &n_sel);
if (status == PRO_TK_NO_ERROR)
{
  status = ProSelectionPoint3dGet(sel[0], refPoint);
  status = ProSelectionViewGet(sel[0], &parentView);
  //Screen coordinate system，注意没有做组件到装配体的变换
  status = _coordsolidtoScreen(parentView, refPoint, refPointScreen);
  //样条曲线四个点为上下左右各偏移20四个点作为圆的内接正方形
  status = ProArrayAlloc(0, sizeof(ProPoint3d), 1, (ProArray *)&pnt_arr);

  refPointScreen[0] -= 20;
  ProArrayObjectAdd((ProArray *)&pnt_arr, PRO_VALUE_UNUSED, 1, refPointScreen);

  refPointScreen[0] += 20;
  refPointScreen[1] -= 20;
  ProArrayObjectAdd((ProArray *)&pnt_arr, PRO_VALUE_UNUSED, 1, refPointScreen);

  refPointScreen[0] += 20;
  refPointScreen[1] += 20;
  ProArrayObjectAdd((ProArray *)&pnt_arr, PRO_VALUE_UNUSED, 1, refPointScreen);
  
  refPointScreen[0] -= 20;
  refPointScreen[1] += 20;
  ProArrayObjectAdd((ProArray *)&pnt_arr, PRO_VALUE_UNUSED, 1, refPointScreen);
  
  status = ProArraySizeGet((ProArray)pnt_arr, &np);

  if (status != PRO_TK_NO_ERROR || np == 0)
    return PRO_TK_BAD_CONTEXT;

  status = ProArrayAlloc(0, sizeof(ProPoint3d), 1, (ProArray *)&p_tan);
  status = ProArrayAlloc(0, sizeof(double), 1, (ProArray *)&par_arr);
  tan_arr = (ProPoint3d *)calloc(np, sizeof(ProPoint3d));
  tan_arr[0][0] = pnt_arr[1][0] - pnt_arr[0][0];
  tan_arr[0][1] = 2 * pnt_arr[1][1] - pnt_arr[2][1] - pnt_arr[0][1];
  tan_arr[np - 1][0] = -(pnt_arr[np - 2][0] - pnt_arr[np - 1][0]);
  tan_arr[np - 1][1] = -(2 * pnt_arr[np - 2][1] - pnt_arr[np - 3][1] - pnt_arr[np - 1][1]);

  for (n = 1; n < np - 1; n++)
  {
    tan_arr[n][0] = pnt_arr[n + 1][0] - pnt_arr[n - 1][0];
    tan_arr[n][1] = pnt_arr[n + 1][1] - pnt_arr[n - 1][1];
  }
  for (n = 0; n < np; n++)
  {
    len = (tan_arr[n][0] * tan_arr[n][0]) + (tan_arr[n][1] * tan_arr[n][1]);
    len = sqrt(len);
    tan_arr[n][0] /= len;
    tan_arr[n][1] /= len;
    status = ProArrayObjectAdd((ProArray *)&p_tan, PRO_VALUE_UNUSED, 1, tan_arr[n]);
  }
  angle = 0.0;
  status = ProArrayObjectAdd((ProArray *)&par_arr, PRO_VALUE_UNUSED, 1, &angle);
  for (n = 1; n < np; n++)
  {
    ProUtilVectorDiff(pnt_arr[n], pnt_arr[n - 1], chord);
    angle = ProUtilVectorLength(chord) + par_arr[n - 1];
    status = ProArrayObjectAdd((ProArray *)&par_arr, PRO_VALUE_UNUSED, 1, &angle);
  }
  status = ProSplinedataInit(par_arr, pnt_arr, p_tan, np, &crv_data);

  //根据实际计算调整，这里做死了
  refPointScreen[0] += 500;
  refPointScreen[1] -= 100;

  status = ProDrawingViewDetailCreate(drawing, parentView, sel[0], &crv_data, refPointScreen, &detailedView);
  status = _setDisplayStyle(drawing, detailedView, PRO_DISPSTYLE_HIDDEN_LINE);
  status = ProDwgSheetRegenerate(drawing, sheet);

  status = ProArrayFree((ProArray *)&p_tan);
  status = ProArrayFree((ProArray *)&par_arr);
  status = ProArrayFree((ProArray *)&pnt_arr);
}
```

创建辅助视图效果如下图所示：

![image](https://github.com/Mountains-and-rivers/creo-second-dev/blob/main/images/4.gif)

6.3.6 旋转剖视图

```
drawing = (ProDrawing)mdl;
status = ProDrawingCurrentsolidGet(drawing, &solid);
status = ProDrawingCurrentSheetGet(drawing, &sheet);
//选择视图里面已有的旋转视图获取信息
status = ProSelect((char *)"dwg_view", 1, NULL, NULL, NULL, NULL, &sel1, &n_sel);
status = ProSelectionViewGet(sel1[0],&revolveView);
status = ProDrawingViewRevolveInfoGet(drawing,revolveView,&xsec,sel1,point);

status = ProSelect((char *)"dwg_view", 1, NULL, NULL, NULL, NULL, &sel, &n_sel);
if (status == PRO_TK_NO_ERROR)
{
  //如果直接都使用ProDrawingViewRevolveInfoGet得到的参数，以下函数返回-2
  //使用ProSelect((char *)"dwg_view"返回的视图则下面函数返回值为PRO_TK_NO_ERROR，但Creo提示"This view has been frozen because of illegal view instructions."
  status = ProDrawingViewRevolveCreate(drawing, NULL, *sel1, point, &revolveView);
  status = _setDisplayStyle(drawing, revolveView, PRO_DISPSTYLE_HIDDEN_LINE);
  status = ProDwgSheetRegenerate(drawing, sheet);
}
```

完整代码参考：https://github.com/slacker-HD/creo_toolkit
