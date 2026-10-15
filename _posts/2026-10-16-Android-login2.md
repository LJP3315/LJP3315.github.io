---
title: "安卓Activity生成注册"
date: 2026-10-16
categories: school
tags: Android
---





```kotlin
// MainActivity.kt

package com.example.exp5

import android.app.Activity
import android.content.Intent
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.rememberLauncherForActivityResult
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.activity.result.contract.ActivityResultContracts
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.Button
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.OutlinedTextField
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.unit.dp
import com.example.exp5.ui.theme.Exp5Theme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            Exp5Theme {
                Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->
                    MainScreen(modifier = Modifier.padding(innerPadding))
                }
            }
        }
    }
}

@Composable
fun MainScreen(modifier: Modifier = Modifier) {
    val context = LocalContext.current

    var username by remember { mutableStateOf("") }
    var password by remember { mutableStateOf("") }
    var address by remember { mutableStateOf("未选择") }

    // 实验内容二 步骤1：通过 rememberLauncherForActivityResult() 注册结果接收器
    val addressLauncher = rememberLauncherForActivityResult(
        contract = ActivityResultContracts.StartActivityForResult()
    ) { result ->
        // 步骤3：在 onResult 回调中获取返回的 ActivityResult，读取 Intent 携带的数据
        if (result.resultCode == Activity.RESULT_OK) {
            address = result.data?.getStringExtra("address") ?: "未选择"
        }
    }

    Column(
        modifier = modifier
            .fillMaxSize()
            .padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(16.dp)
    ) {
        Text(
            text = "用户注册",
            style = MaterialTheme.typography.headlineMedium
        )

        OutlinedTextField(
            value = username,
            onValueChange = { username = it },
            label = { Text("用户名") },
            singleLine = true,
            modifier = Modifier.fillMaxWidth()
        )

        OutlinedTextField(
            value = password,
            onValueChange = { password = it },
            label = { Text("密码") },
            singleLine = true,
            modifier = Modifier.fillMaxWidth()
        )

        Row(
            modifier = Modifier.fillMaxWidth(),
            verticalAlignment = Alignment.CenterVertically,
            horizontalArrangement = Arrangement.spacedBy(12.dp)
        ) {
            Button(onClick = {
                // 步骤1：使用 launcher.launch(intent) 启动 AddressSelectedActivity
                addressLauncher.launch(
                    Intent(context, AddressSelectedActivity::class.java)
                )
            }) {
                Text("地址选择")
            }
            Text(text = "当前地址：$address")
        }

        Spacer(modifier = Modifier.height(8.dp))

        Button(
            onClick = {
                // 实验内容一：MainActivity 发送数据的四个步骤
                // 1. 建立 Intent
                val intent = Intent(context, ResultActivity::class.java)
                // 2. 新建 Bundle，并设置数据
                val bundle = Bundle().apply {
                    putString("username", username)
                    putString("password", password)
                    putString("address", address)
                }
                // 3. 使用 Intent 携带 Bundle
                intent.putExtras(bundle)
                // 4. 启动 Activity（StartActivity 形式）
                context.startActivity(intent)
            },
            modifier = Modifier.fillMaxWidth()
        ) {
            Text("下一步")
        }
    }
}

```



```kotlin
// ResultActivity.kt

package com.example.exp5

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import com.example.exp5.ui.theme.Exp5Theme

class ResultActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()

        // 实验内容一：ResultActivity 获取数据的三个步骤
        // 1. 获得MainActivity传递来的Intent
        val intent = intent
        // 2. 获得Intent携带的Bundle
        val bundle = intent.extras
        // 3. 通过Bundle获得对应数据
        val username = bundle?.getString("username") ?: ""
        val password = bundle?.getString("password") ?: ""
        val address = bundle?.getString("address") ?: ""

        setContent {
            Exp5Theme {
                Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->
                    ResultScreen(
                        username = username,
                        password = password,
                        address = address,
                        modifier = Modifier.padding(innerPadding)
                    )
                }
            }
        }
    }
}

@Composable
fun ResultScreen(
    username: String,
    password: String,
    address: String,
    modifier: Modifier = Modifier
) {
    Column(
        modifier = modifier
            .fillMaxSize()
            .padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(12.dp)
    ) {
        Text(
            text = "您的注册信息",
            style = MaterialTheme.typography.headlineMedium
        )
        InfoRow(label = "用户名", value = username)
        InfoRow(label = "密码", value = password)
        InfoRow(label = "地址", value = address)
    }
}

@Composable
fun InfoRow(label: String, value: String) {
    Row(
        modifier = Modifier.fillMaxWidth(),
        verticalAlignment = Alignment.CenterVertically
    ) {
        Text(
            text = "$label：",
            style = MaterialTheme.typography.bodyLarge
        )
        Text(
            text = value,
            style = MaterialTheme.typography.bodyLarge
        )
    }
}
```



```kotlin
// AddressSelectedAcitivity.kt

package com.example.exp5

import android.app.Activity
import android.content.Intent
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.HorizontalDivider
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import com.example.exp5.ui.theme.Exp5Theme

class AddressSelectedActivity : ComponentActivity() {

    private val addressList = listOf(
        "北京市", "上海市", "广州市", "深圳市",
        "杭州市", "成都市", "武汉市", "西安市"
    )

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            Exp5Theme {
                Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->
                    AddressListScreen(
                        addresses = addressList,
                        modifier = Modifier.padding(innerPadding),
                        onAddressClick = { selected ->
                            // 实验内容二 步骤2：将返回数据放入 Intent，
                            // 通过 setResult() 设置返回结果，并调用 finish() 关闭当前 Activity
                            val intent = Intent()
                            intent.putExtra("address", selected)
                            setResult(Activity.RESULT_OK, intent)
                            finish()
                        }
                    )
                }
            }
        }
    }
}

@Composable
fun AddressListScreen(
    addresses: List<String>,
    modifier: Modifier = Modifier,
    onAddressClick: (String) -> Unit
) {
    Column(
        modifier = modifier
            .fillMaxSize()
            .padding(vertical = 8.dp),
        verticalArrangement = Arrangement.spacedBy(0.dp)
    ) {
        Text(
            text = "请选择地址",
            style = MaterialTheme.typography.headlineSmall,
            modifier = Modifier.padding(horizontal = 16.dp, vertical = 12.dp)
        )
        addresses.forEach { address ->
            Text(
                text = address,
                style = MaterialTheme.typography.bodyLarge,
                modifier = Modifier
                    .fillMaxWidth()
                    .clickable { onAddressClick(address) }
                    .padding(horizontal = 16.dp, vertical = 16.dp)
            )
            HorizontalDivider()
        }
    }
}
```

```xml
AndroidManifest

<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.Exp5">
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:label="@string/app_name"
            android:theme="@style/Theme.Exp5"
            android:windowSoftInputMode="adjustResize">
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

