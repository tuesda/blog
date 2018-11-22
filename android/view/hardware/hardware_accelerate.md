android硬件加速分为四个层级：

1. Application 

	```
	<application android:hardwareAccelerated="true" ...>
	```
2. Window

	```
	<application android:hardwareAccelerated="true">  
	<activity ... />  
	<activity android:hardwareAccelerated="false" />  
	</application>  

	```
3. Activity

	```
	getWindow().setFlags(
		WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED,
	 	WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED
	); 
	```
4. View

	```
	myView.setLayerType(View.LAYER_TYPE_SOFTWARE, null);  
	```
	
	
#####注意事项:
* 当Application硬件加速全局打开的时候，就是上述第一种方式。通过window manager添加的view在有的机型上硬件加速没有打开，需要通过第二种方式在专门打开window的硬件加速