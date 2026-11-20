# 4G_NET
تطبيق شبكه فورجي نت - تعز الظهرة 
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.fourgnet.hotspot">

    <!-- صلاحيات الاتصال بالشبكة -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="شبكة فورجي نت"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.AppCompat.NoActionBar"
        android:usesCleartextTraffic="true">

        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:screenOrientation="portrait">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="#121622">

    <!-- شريط علوي باسم الشبكة -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="54dp"
        android:background="#1A1F2E"
        android:gravity="center"
        android:paddingHorizontal="16dp">

        <TextView
            android:id="@+id/tvNetworkTitle"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="بوابة شبكة أمجد نت"
            android:textColor="#F5B041"
            android:textSize="18sp"
            android:textStyle="bold" />
    </LinearLayout>

    <!-- شريط تحميل رفيع -->
    <ProgressBar
        android:id="@+id/progressBar"
        style="?android:attr/progressBarStyleHorizontal"
        android:layout_width="match_parent"
        android:layout_height="3dp"
        android:max="100"
        android:progressTint="#F5B041"
        android:visibility="gone" />

    <!-- متصفح صفحة الهوت سبوت -->
    <WebView
        android:id="@+id/webView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />

    <!-- شريط التحكم السفلي السريع -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="60dp"
        android:background="#1A1F2E"
        android:orientation="horizontal"
        android:weightSum="3">

        <Button
            android:id="@+id/btnStatus"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:background="?attr/selectableItemBackground"
            android:text="حالة الرصيد"
            android:textColor="#FFFFFF"
            android:textSize="13sp" />

        <Button
            android:id="@+id/btnLogin"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:background="?attr/selectableItemBackground"
            android:text="تسجيل كرت"
            android:textColor="#F5B041"
            android:textStyle="bold"
            android:textSize="13sp" />

        <Button
            android:id="@+id/btnRefresh"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:background="?attr/selectableItemBackground"
            android:text="تحديث"
            android:textColor="#FFFFFF"
            android:textSize="13sp" />
    </LinearLayout>

</LinearLayout>
package com.amjadnet.hotspot

import android.annotation.SuppressLint
import android.content.Context
import android.net.ConnectivityManager
import android.net.Network
import android.net.NetworkCapabilities
import android.net.NetworkRequest
import android.os.Bundle
import android.view.View
import android.webkit.*
import android.widget.Button
import android.widget.ProgressBar
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {

    private lateinit var webView: WebView
    private lateinit var progressBar: ProgressBar

    // روابط بوابة الهوت سبوت الخاصة بشبكتك
    private val statusUrl = "http://172.20.1.1/status"
    private val loginUrl = "http://172.20.1.1/login"
    private val homeUrl = "http://172.20.1.1/index.html"

    @SuppressLint("SetJavaScriptEnabled")
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        webView = findViewById(R.id.webView)
        progressBar = findViewById(R.id.progressBar)

        val btnStatus: Button = findViewById(R.id.btnStatus)
        val btnLogin: Button = findViewById(R.id.btnLogin)
        val btnRefresh: Button = findViewById(R.id.btnRefresh)

        // إجبار التطبيق على الاتصال عبر الواي فاي المحلي حتى لو بيانات الهاتف شغالة
        bindToWifiNetwork()

        // إعدادات WebView
        webView.settings.apply {
            javaScriptEnabled = true
            domStorageEnabled = true
            cacheMode = WebSettings.LOAD_NO_CACHE
            useWideViewPort = true
            loadWithOverviewMode = true
        }

        webView.webChromeClient = object : WebChromeClient() {
            override fun onProgressChanged(view: WebView?, newProgress: Int) {
                if (newProgress < 100) {
                    progressBar.visibility = View.VISIBLE
                    progressBar.progress = newProgress
                } else {
                    progressBar.visibility = View.GONE
                }
            }
        }

        webView.webViewClient = object : WebViewClient() {
            override fun onReceivedError(
                view: WebView?,
                request: WebResourceRequest?,
                error: WebResourceError?
            ) {
                if (request?.isForMainFrame == true) {
                    val errorHtml = """
                        <html dir="rtl">
                        <body style="background:#121622;color:white;font-family:sans-serif;text-align:center;padding-top:100px;">
                            <h2 style="color:#F5B041;">تعذر الوصول إلى بوابة الشبكة</h2>
                            <p>تأكد من اتصالك بشبكة الواي فاي الخاصة بنا ثم اضغط على تحديث.</p>
                        </body>
                        </html>
                    """.trimIndent()
                    webView.loadDataWithBaseURL(null, errorHtml, "text/html", "UTF-8", null)
                }
            }
        }

        // تحميل الصفحة الرئيسية للهوت سبوت
        webView.loadUrl(homeUrl)

        // أزرار الشريط السفلي
        btnStatus.setOnClickListener { webView.loadUrl(statusUrl) }
        btnLogin.setOnClickListener { webView.loadUrl(loginUrl) }
        btnRefresh.setOnClickListener { webView.reload() }
    }

    private fun bindToWifiNetwork() {
        val connectivityManager =
            getSystemService(Context.CONNECTIVITY_SERVICE) as ConnectivityManager
        val request = NetworkRequest.Builder()
            .addTransportType(NetworkCapabilities.TRANSPORT_WIFI)
            .build()

        connectivityManager.registerNetworkCallback(
            request,
            object : ConnectivityManager.NetworkCallback() {
                override fun onAvailable(network: Network) {
                    connectivityManager.bindProcessToNetwork(network)
                }
            }
        )
    }

    override fun onBackPressed() {
        if (webView.canGoBack()) {
            webView.goBack()
        } else {
            super.onBackPressed()
        }
    }
}
