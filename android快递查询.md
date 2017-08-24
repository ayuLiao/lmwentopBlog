---
title: android快递查询
date: 2016-06-14 15:02:25
tags: android
---
## 简介
项目本身的意义不大，因为没有人会去记忆快递单号，如果有快递，一般都是通过电商购物平台的app快递查看的功能直接查看，而不会自己手动去查看。项目基于android，利用webview控件来实现，十分简单。

## 思路
利用Edittext来接收用户输入的快递公司名和快递的单号，通过Intent传递给另一个Activity，该Activity上声明了webView，webView利用快递公司名和快递单号构建出来的url直接访问进行查询！

通过一个url接口来查找快递：http://m.kuaidi100.com/index_all.html?type=[快递公司]&postid=[快递单号]

布局：
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <ImageView
	    android:id="@+id/img"
	    android:layout_width="wrap_content"
	    android:layout_height="wrap_content"
	    android:src="@drawable/aim" 
	    android:layout_centerHorizontal="true"
	    android:layout_marginTop="50px"/>
	<EditText 
	    android:id="@+id/company"
	    android:layout_width="match_parent"
	    android:layout_height="wrap_content"
	    android:hint="输入快递公司名称"
	    android:textSize="50px"
	    android:layout_marginTop="20px"
	    android:layout_below="@+id/img"/>
	<EditText 
	    android:id="@+id/number"
	    android:layout_width="match_parent"
	    android:layout_height="wrap_content"
	    android:hint="输入快递单号"
	    android:textSize="50px"
	    android:layout_marginTop="20px"
	    android:layout_below="@+id/company"/>
	
	<Button 
	    android:id="@+id/showWuli"
	    android:layout_width="match_parent"
	    android:layout_height="wrap_content"
	    android:text="查询"
	    android:textSize="50px"
	    android:layout_below="@+id/number"
	    android:layout_marginTop="20px"/>
</RelativeLayout>
```
![输入布局](http://obfs4iize.bkt.clouddn.com/android%E5%BF%AB%E9%80%92%E6%9F%A5%E8%AF%A2_1.png)

## 程序：

MainActivity.class
```Java
package com.liao.showwuli;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.EditText;

public class MainActivity extends Activity {

	private EditText company;//快递公司名
	private EditText number;//快递单号
	private Button showWuli;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.fragment_main);

		init();

		showWuli.setOnClickListener(new OnClickListener() {

			@Override
			public void onClick(View arg0) {
				Intent intent = new Intent(MainActivity.this,webView.class);
				intent.putExtra("company", company.getText().toString());//传递company信息
				intent.putExtra("number", number.getText().toString());
				startActivity(intent);
			}
		});

	}

	private void init() {
		company = (EditText) findViewById(R.id.company);
		number = (EditText) findViewById(R.id.number);
		showWuli = (Button) findViewById(R.id.showWuli);
	}



}

```


webView.class
```Java
package com.liao.showwuli;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.util.Log;
import android.webkit.WebView;
import android.webkit.WebViewClient;

public class webView extends Activity{
	
	private WebView webView;
	private String url;
	private String company;
	private String number;
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.web);
		
		init();
		
		Intent getIntent = getIntent();
		company = getIntent.getStringExtra("company");
		number = getIntent.getStringExtra("number");
		
		url = "http://m.kuaidi100.com/index_all.html?type="+company+"&postid="+number;
		
		webView.getSettings().setJavaScriptEnabled(true);
		webView.loadUrl(url);
		
		webView.setWebViewClient(new WebViewClient(){
			@Override
			public boolean shouldOverrideUrlLoading(WebView view, String url) {
				view.loadUrl(url);
				return super.shouldOverrideUrlLoading(view, url);
			}
			
		});
		
	}
	private void init() {
		webView = (WebView) findViewById(R.id.wuli);
	}
}

```


这样就可以查询到快递了
![查询结果图](http://obfs4iize.bkt.clouddn.com/android%E5%BF%AB%E9%80%92%E6%9F%A5%E8%AF%A2_2.png)

如果需要，伸手党可以在[物流眼](https://github.com/ayuLiao/wulieye)中下载到
该例子使用ecplise开发，sdk为19
