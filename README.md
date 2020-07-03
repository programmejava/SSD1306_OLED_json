<h1 id="目录">目录</h1>  

* [目录](#目录) 
* [简介](#简介)  
* [配置文件](#配置文件)  
	* [开始运行](#开始运行)  
	* [json简介](#json简介)  
	* [整体结构](#整体结构)  
	* [软硬件设置](#软硬件设置)  
	* [页面数据](#页面数据)  
		* [页面设置](#页面设置)  
		* [显示数据](#显示数据)  
	* [显示元素](#显示元素)  
		* [内置基础选项](#内置基础选项)  
		* [点](#点)  
		* [圆](#圆)  
		* [矩形](#矩形)  
		* [圆角矩形](#圆角矩形)  
		* [三角形](#三角形)  
		* [数字](#数字)  
		* [字符串](#字符串)  
		* [shell命令返回值](#shell命令返回值)  
* [引用](#引用)  

---

<h1 id="简介">简介</h1>  

嵌入式linux驱动SSD1306芯片的OLED，采用json文件配置显示内容  

---
<h1 id="配置文件">配置文件</h1>  

配置文件采用json格式，使用cJSON解析，语法遵循json语法

<h2 id="开始运行">开始运行</h2>  

**方法1：**
将config.json和二进制ssd放入同一目录，终端进入该目录执行
```
./ssd
```
**方法2**：
将config.json放入`/xxx`目录，终端进入`ssd`所在目录执行  
```
./ssd -c /xxx/config.json
```

<h2 id="json简介">json简介</h2>

* JSON(JavaScript Object Notation, JS 对象简谱) 是一种轻量级的数据交换格式  
* 其主要元素为`健`、`值`、`符号`  
* 基本语法规则:  
	1. 数据在`健/值`对中  
	2. 数据由`逗号`分隔  
	3. `大括号`保存`对象`  
	4. `中括号`保存`数组`  
* `健`必须是`字符串`，且用`""`引起来  
* `值`可以是  
	1. `数字`（整数或浮点数）  
	3. `字符串`（在双引号中）  
	4. `逻辑值`（true 或 false）  
	5. `数组`（在中括号中）  
	6. `对象`（在大括号中）  
	7. `null`  
注：在本程序配置文件中只采用了`数字`、`字符串`、`数组`、`对象`  

<h2 id="整体结构">整体结构</h2>  

本程序使用分页的方式显示各种内容，json表现为多个页面数据在同一级。  
配置文件整体结构如下：  
```
{
	"seting":{},
	"eg1":{},
	"eg2":{}
}
```
|键|值类型|说明|
|:----:|:----:|:----:|
|`"seting"`|`对象`|[软硬件设置](#软硬件设置)，包含了分辨率、地址以及驱动文件等信息|
|`"eg1"`、`"eg2"`|`对象`|[页面数据](#页面数据),包含了页面相关数据，名称可以自定义。|

<h2 id="软硬件设置">软硬件设置</h2>  

```
"seting":{
	"pixel":12864,
	"dev":"/dev/i2c-0",
	"addr":60
}
```
|键|值类型|取值范围|默认值|说明|
|:----:|:----:|:----:|:----:|:----:|
|`"pixel"`|`数字`|`12864`、`12832`|`12864`|OLED分辨率
|`"dev"`|`字符串`|i2c驱动范围|`"/dev/i2c-0"`|i2c驱动路径
|`"addr"`|`数字`|i2c地址范围|`60`|OLED在i2c总线的地址，<br>`60`即十六进制`0x3c`（json不支持十六进制表示）

注：在配置文件中，可以省略软硬件设置`"seting":{}`  

<h2 id="页面数据">页面数据</h2>  

```
"eg1":{
	"seting":{},
	"display":[]
}
```
|键|值类型|说明|
|:----:|:----:|:----:|
|`"seting"`|`对象`|[页面设置](#页面设置)，包含了刷新周期、显示总时长等信息，每个页面的`"seting"`不可省略！|
|`"display"`|`数组`|[显示数据](#显示数据)，包含了频幕上的显示元素，<br>数组为`对象`数组，每一个`对象`都是页面上显示的一个元素。<br>`"display"`可省略，省略后OLED显示空白页面(什么都不显示)|

注：页面数据`"eg1"`名称可自定义，符合json语法规则即可

<h3 id="页面设置">1. 页面设置</h3>  

```
"seting":{
	"cycle":1,
	"time":1,
	"page":0
},
```
此设置包含在页面之下，和显示数据`"display"`同级，主要设置本页面属性
|键|值类型|取值范围|默认值|说明|
|:----:|:----:|:----:|:----:|:----:|
|"cycle"|`数字`|`0`至`30000`|`1`|显示周期，时间到则刷新屏幕，<br>`实际周期 = cycle * 100ms`，建议`"cycle"`取值`5`|
|"time"|`数字`|`0`至`30000`|`1`|显示时长，一个页面显示时间，<br>`实际时间 = time * 100ms`，建议取值`50`至`100`|
|"page"|`数字`|`0`至`30000`|`0`|页码，决定显示顺序，<br>值为`0`时则不显示该页面，<br>两个页面页码相同时只显示配置文件靠前的页面，<br>中间断码则页面显示到断码页为止，后面页则不显示|

`"cycle"`必须小于`"time"`，当`"cycle"`大于`"time"`时页面将不刷新

<h3 id="显示数据">2. 显示数据</h3>  

```
"display":[
	{"type":"eg1","x0":0,"y0":0 ... },
	{"type":"eg2","x0":0,"y0":0 ... }
]
```
|键|值类型|取值范围|默认值|说明|
|:----:|:----:|:----:|:----:|:----:|
|`"display"`|`数组`|[显示元素](#显示元素)|NONE|显示内容，属于[页面数据](#页面数据)，<br>决定OLED显示内容，由多个元素构成。<br>内容可为空，内容为空时显示空白页面(什么都不显示)|

<h2 id="显示元素">显示元素</h2>  

显示元素是[显示数据](#显示数据)`"display":[]`数组的值，决定OLED显示的具体元素，比如点、线、字符串等,元素包含各种属性，其中`"type"`、`"x0"`、`"y0"`、`"color"`、`"en"`属性必不可少
|键|值类型|取值范围|默认值|说明|
|:----:|:----:|:----:|:----:|:----:|
|`"type"`|`字符串`|各个元素名|NONE|元素类型，告诉软件该显示什么|
|`"x0"`|`数字`|`0`至`最大横向像素减1`|`0`|起始横向坐标，如果是12864，则最大值为127|
|`"y0"`|`数字`|`0`至`最大纵向像素减1`|`0`|起始纵向坐标，如果是12864，则最大值为63|
|`"color"`|`数字`|`0`至`2`|`0`|显示颜色，决定元素显示颜色，`0`:黑;`1`:白;`2`:反色|
|`"en"`|`数字`|`0`或`1`|`0`|元素使能，`0`:不显示;`0`:显示|

<h3 id="内置基础选项">1. 内置基础选项</h3>  

```
```

<h3 id="点">3. 点</h3> 

```
{"type":"pixel", "x0":0, "y0":0, "color":0, "en":0}
```
|键|值类型|取值范围|默认值|说明|
|:----:|:----:|:----:|:----:|:----:|
|`"type"`|`字符串`|`"pixel"`|NONE|线元素名称，只能是`"pixel"`|

<h3 id="线">2. 线</h3>  

```
{ "type":"line", "x0":0, "y0":0, "x1":0, "y1":0, "color":0, "en":0}
```
|键|值类型|取值范围|默认值|说明|
|:----:|:----:|:----:|:----:|:----:|
|`"type"`|`字符串`|`"line"`|NONE|线元素名称，只能是`"line"`|
|`"x1"`|`数字`|`0`至`最大横向像素 - 1`|`0`|起始横向坐标，如果是12864，则最大值为127|
|`"y1"`|`数字`|`0`至`最大纵向像素 - 1`|`0`|起始纵向坐标，如果是12864，则最大值为63|

其他属性说明请看[显示元素](#显示元素)

<h3 id="圆">4. 圆</h3>  

```
{ "type":"circle", "x0":0, "y0":0, "r":0, "fill":0, "color":0, "en":0}
```
|键|值类型|取值范围|默认值|说明|
|:----:|:----:|:----:|:----:|:----:|
|`"type"`|`字符串`|`"circle"`|NONE|圆元素名称，只能是`"circle"`|
|`"r"`|`数字`|`0`至`127`|`0`|圆半径，单位：像素|
|`"fill"`|`数字`|`0`或`1`|`0`|填充使能，0:失能;1:使能|

其他属性说明请看[显示元素](#显示元素)

<h3 id="矩形">5. 矩形</h3>  

```
{ "type":"rect", "x0":0, "y0":0, "w":0, "h":0, "fill":0, "color":0, "en":0}
```
|键|值类型|取值范围|默认值|说明|
|:----:|:----:|:----:|:----:|:----:|
|`"type"`|`字符串`|`"rect"`|NONE|矩形元素名称，只能是`"rect"`|
|`"w"`|`数字`|`0`至`127`|`0`|矩形宽，单位：像素|
|`"h"`|`数字`|`0`或`127`|`0`|矩形高，单位：像素|
|`"fill"`|`数字`|`0`或`1`|`0`|填充使能，0:失能;1:使能|

其他属性说明请看[显示元素](#显示元素)

<h3 id="圆角矩形">6. 圆角矩形</h3>  

```
{"type":"r_rect", "x0":0, "y0":0, "w":0, "h":0, "r":0, "fill":0, "color":0, "en":0}
```
|键|值类型|取值范围|默认值|说明|
|:----:|:----:|:----:|:----:|:----:|
|`"type"`|`字符串`|`"r_rect"`|NONE|圆角矩形元素名称，只能是`"r_rect"`|
|`"w"`|`数字`|`0`至`127`|`0`|矩形宽，单位：像素|
|`"h"`|`数字`|`0`或`127`|`0`|矩形高，单位：像素|
|`"r"`|`数字`|`0`至`127`|`0`|圆角半径，单位：像素|
|`"fill"`|`数字`|`0`或`1`|`0`|填充使能，0:失能;1:使能|

其他属性说明请看[显示元素](#显示元素)

<h3 id="三角形">7. 三角形</h3>  

```
{"type":"trle", "x0":0, "y0":0, "x1":0, "y1":0, "x2":0, "y2":0, "fill":0, "color":0, "en":0}
```
|键|值类型|取值范围|默认值|说明|
|:----:|:----:|:----:|:----:|:----:|
|`"type"`|`字符串`|`"trle"`|NONE|三角形元素名称，只能是`"trle"`|
|`"x1"`|`数字`|`0`至`最大横向像素 - 1`|`0`|起始横向坐标，如果是12864，则最大值为127|
|`"y1"`|`数字`|`0`至`最大纵向像素 - 1`|`0`|起始纵向坐标，如果是12864，则最大值为63|
|`"x2"`|`数字`|`0`至`最大横向像素 - 1`|`0`|起始横向坐标，如果是12864，则最大值为127|
|`"y2"`|`数字`|`0`至`最大纵向像素 - 1`|`0`|起始纵向坐标，如果是12864，则最大值为63|
|`"fill"`|`数字`|`0`或`1`|`0`|填充使能，0:失能;1:使能|

其他属性说明请看[显示元素](#显示元素)

<h3 id="数字">8. 数字</h3>  

```
{"type":"num", "func":"ui", "data":0, "base":0, "x0":0, "y0":0, "size":0, "color":0, "en":0}
```
虽然数字是个可用选项，但是由于数字的种类多样，可能产生不稳定因素，所以建议使用[字符串](#字符串)代替
|键|值类型|取值范围|默认值|说明|
|:----:|:----:|:----:|:----:|:----:|
|`"type"`|`字符串`|`"num"`|NONE|数字元素名称，只能是`"num"`|
|`"func"`|`字符串`|`"ui"`、`"fl"`|NONE|数字种类，`ui`:无符号整形，即unsigned int;`fl `:浮点数|
|`"data"`|`数字`|无符号整形:`0`至`32767`,<br>浮点数:double范围|`0`|待显示数字，不建议使用，建议[字符串](#字符串)代替|
|`"base"`|`数字`|`0`至`30000`|`0`|显示位数，如果是浮点数，小数点也算在内|
|`"size"`|`数字`|`0`至`255`|`0`|字体放大比例，虽然上限可以到255，<br>但是没意义且取值太大不保证程序稳定性|

其他属性说明请看[显示元素](#显示元素)

<h3 id="字符串">9. 字符串</h3>  

```
{"type":"str", "data":"String:", "x0":0, "y0":0, "size":0, "color":0, "en":0}
```
|键|值类型|取值范围|默认值|说明|
|:----:|:----:|:----:|:----:|:----:|
|`"type"`|`字符串`|`"str"`|NONE|字符串元素名称，只能是`"str"`|
|`"data"`|`字符串`|字符串|NONE|待显示字符串，<br>特殊符号需要转意，<br>具体查看json语法|
|`"size"`|`数字`|`0`至`255`|`0`|字体放大比例，虽然上限可以到255，<br>但是没意义且取值太大不保证程序稳定性|

其他属性说明请看[显示元素](#显示元素)

<h3 id="shell命令返回值">10. shell命令返回值</h3>  

```
{"type":"cmd", "data":"ifconfig", "x0":0, "y0":0, "base":0, "size":0, "color":0, "en":0}
```
|键|值类型|取值范围|默认值|说明|
|:----:|:----:|:----:|:----:|:----:|
|`"type"`|`字符串`|`"cmd"`|NONE|shell命令元素名称，只能是`"cmd"`|
|`"data"`|`字符串`|字符串|NONE|待执行命令，<br>特殊符号需要转意，<br>具体查看json语法|
|`"base"`|`数字`|`0`至`30000`|`0`|显示位数|
|`"size"`|`数字`|`0`至`255`|`0`|字体放大比例，虽然上限可以到255，<br>但是没意义且取值太大不保证程序稳定性|

其他属性说明请看[显示元素](#显示元素)

<h1 id="引用">引用</h1>  

---
 * json解析库cJSON:[DaveGamble/cJSON](https://github.com/DaveGamble/cJSON)  
 * SSD1306驱动库:[deeplyembeddedWP/SSD1306-OLED-display-driver-for-BeagleBone](https://github.com/deeplyembeddedWP/SSD1306-OLED-display-driver-for-BeagleBone)  
