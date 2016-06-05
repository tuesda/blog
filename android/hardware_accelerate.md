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