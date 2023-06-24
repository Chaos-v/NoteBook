*@DATE：2023/02/28 10:04:17*

---

1. "Win + R" 打开运行

2. 输入 “regedit” 打开注册表

3. 根据使用的版本找到以下注册表子项之一：

	- PowerPoint 2016、PowerPoint 2019、Microsoft 365 专属 PowerPoint
	
	```
		HKEY_CURRENT_USER\Software\Microsoft\Office\16.0\PowerPoint\Options
	```

	- PowerPoint 2013
		
	```
		HKEY_CURRENT_USER\Software\Microsoft\Office\15.0\PowerPoint\Options
	```
		
	- PowerPoint 2010

	```
		HKEY_CURRENT_USER\Software\Microsoft\Office\14.0\PowerPoint\Options
	```

	- PowerPoint 2007

	```
		HKEY_CURRENT_USER\Software\Microsoft\Office\12.0\PowerPoint\Options
	```

	- PowerPoint 2003
	
	```
		HKEY_CURRENT_USER\Software\Microsoft\Office\11.0\PowerPoint\Options
	```
4. 单击“选项”子项，指向“编辑”菜单上的“新建”，然后选择“DWORD (32 位)值”

5. 输入“ExportBitmapResolution”，然后按 Enter 键。

6. 修改“ExportBitmapResolution”的值，在“编辑 DWORD 值”对话框中选择 **“十进制”** ，在“数值数据”框中，输入分辨率“300”。 或使用下表中的参数。

<table>
    <thead>
        <tr>
            <th>十进制值</th>
            <th>全屏像素（水平 × 垂直）</th>
            <th>宽屏像素（水平 + 垂直）</th>
            <th>每英寸点数（水平和垂直）</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>50</td>
            <td>500 × 375</td>
            <td>667 × 375</td>
            <td>50 dpi</td>
        </tr>
        <tr>
            <td>96（默认值）</td>
            <td>960 × 720</td>
            <td>1280 × 720</td>
            <td>96 dpi</td>
        </tr>
        <tr>
            <td>100</td>
            <td>1000 × 750</td>
            <td>1333 × 750</td>
            <td>100 dpi</td>
        </tr>
        <tr>
            <td>150</td>
            <td>1500 × 1125</td>
            <td>2000 × 1125</td>
            <td>150 dpi</td>
        </tr>
        <tr>
            <td>200</td>
            <td>2000 × 1500</td>
            <td>2667 × 1500</td>
            <td>200 dpi</td>
        </tr>
        <tr>
            <td>250</td>
            <td>2500 × 1875</td>
            <td>3333 × 1875</td>
            <td>250 dpi</td>
        </tr>
        <tr>
            <td>300</td>
            <td>3000 × 2250</td>
            <td>4000 × 2250</td>
            <td>300 dpi</td>
        </tr>
    </tbody>
</table>