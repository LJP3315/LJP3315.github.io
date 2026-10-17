---
title: "安卓 Activity生成注册 Intent传递信息"
date: 2026-10-16
categories: school
tags: Android
---

主要记录实验5中出现的Activity之间的跳转，使用Intent在不同界面传输数据



### MainActivity.kt

```kotlin
package com.example.exp5

import android.app.Activity
import android.content.Intent
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.compose.rememberLauncherForActivityResult
import androidx.activity.result.contract.ActivityResultContracts
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.unit.dp

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {
            val context = LocalContext.current

            var username by remember { mutableStateOf("") }
            var password by remember { mutableStateOf("") }
            var address by remember { mutableStateOf("未选择") }

            // 接收地址选择页面返回的数据
            val addressLauncher = rememberLauncherForActivityResult(
                ActivityResultContracts.StartActivityForResult()
            ) { result ->
                if (result.resultCode == Activity.RESULT_OK) {
                    address = result.data?.getStringExtra("address") ?: "未选择"
                }
            }

            Column(
                modifier = Modifier
                    .fillMaxSize()
                    .padding(24.dp),
                verticalArrangement = Arrangement.spacedBy(12.dp)
            ) {
                Text(
                    text = "用户注册",
                    style = MaterialTheme.typography.headlineMedium
                )

                // 输入用户名
                OutlinedTextField(
                    value = username,
                    onValueChange = { username = it },
                    label = { Text("用户名") },
                    modifier = Modifier.fillMaxWidth()
                )

                // 输入密码
                OutlinedTextField(
                    value = password,
                    onValueChange = { password = it },
                    label = { Text("密码") },
                    modifier = Modifier.fillMaxWidth()
                )

                // 跳转到地址选择页面
                Button(
                    onClick = {
                        addressLauncher.launch(
                            Intent(context, AddressSelectedActivity::class.java)
                        )
                    },
                    modifier = Modifier.fillMaxWidth()
                ) {
                    Text("选择地址")
                }

                // 显示当前选择的地址
                Text(text = "当前地址：$address")

                // 跳转到结果页面并传递数据
                Button(
                    onClick = {

                        // 第一步：建立 Intent
                        val intent = Intent(
                            context,
                            ResultActivity::class.java
                        )

                        // 第二步：新建 Bundle，并设置数据
                        val bundle = Bundle()
                        bundle.putString("username", username)
                        bundle.putString("password", password)
                        bundle.putString("address", address)

                        // 第三步：使用 Intent 携带 Bundle
                        intent.putExtras(bundle)

                        // 第四步：启动 ResultActivity
                        context.startActivity(intent)
                    },
                    modifier = Modifier.fillMaxWidth()
                ) {
                    Text("下一步")
                }
            }
        }
    }
}
```

### ResultActivity.kt

```kotlin
package com.example.exp5

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp

class ResultActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // 第一步：获得 MainActivity 传递来的 Intent
        val intent = intent

        // 第二步：获得 Intent 携带的 Bundle
        val bundle = intent.extras

        // 第三步：通过 Bundle 获得对应数据
        val username = bundle?.getString("username") ?: ""
        val password = bundle?.getString("password") ?: ""
        val address = bundle?.getString("address") ?: "未选择"

        setContent {
            ResultScreen(
                username = username,
                password = password,
                address = address
            )
        }
    }
}

@Composable
fun ResultScreen(
    username: String,
    password: String,
    address: String
) {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(24.dp),
        verticalArrangement = Arrangement.spacedBy(12.dp)
    ) {
        Text(
            text = "注册信息",
            style = MaterialTheme.typography.headlineMedium
        )

        Text(text = "用户名：$username")
        Text(text = "密码：$password")
        Text(text = "地址：$address")
    }
}
```

### AddressSelectedAcitivity.kt

```kotlin
package com.example.exp5

import android.app.Activity
import android.content.Intent
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp

// 显示可供选择的地址列表
// 点击地址后，将选中的地址放入Intent
// 使用 setResult 返回数据，并使用 finish 关闭当前页面
class AddressSelectedActivity : ComponentActivity() {

    private val addressList = listOf(
        "北京市", "上海市", "广州市", "深圳市",
        "杭州市", "成都市", "武汉市", "西安市"
    )

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {
            Column(modifier = Modifier.padding(16.dp)) {
                Text("请选择地址")

                addressList.forEach { address ->
                    Text(
                        text = address,
                        modifier = Modifier
                            .fillMaxWidth()
                            .clickable {
                                val intent = Intent()
                                intent.putExtra("address", address)
                                setResult(Activity.RESULT_OK, intent)
                                finish()
                            }
                            .padding(12.dp)
                    )
                }
            }
        }
    }
}
```

### AndroidManifest.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <application
        android:theme="@style/Theme.Exp5"
        android:label="@string/app_name">

        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <activity
            android:name=".AddressSelectedActivity"
            android:exported="false" />
        <activity
            android:name=".ResultActivity"
            android:exported="false" />
    </application>
</manifest>
```

