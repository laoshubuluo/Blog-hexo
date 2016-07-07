title: Android-Code-Volley支持HTTPS
description: Android开发技术点：Volley支持HTTPS
date: 2016/03/13 18:12:12 
categories: Android
tags: [Android,Code,Volley,Network,Https]
---
Volley可以支持HTTPS，但是框架中默认没有加上去，我们可以修改一小部分源码来实现这以功能。

<!--more-->

# 特别感谢 #
文章参考：
> http://blog.csdn.net/llwdslal/article/details/18052723

如有侵权，请与我联系~我会尽快处理并删除侵权内容。

# 正文 #
volley的网络请求 先要通过toolbox包下的Volley.Java生成一个requestQueue.在requestQueue去分发请求，处理请求是使用HttpStack接口来完成的。看下面的代码Volley.java中的newRequestQueueInDisk

	public static RequestQueue newRequestQueueInDisk(Context context, String dir, HttpStack stack) {  
	        File cacheDir = new File(dir, DEFAULT_CACHE_DIR);  
	  
	        String userAgent = "volley/0";  
	        try {  
	            String packageName = context.getPackageName();  
	            PackageInfo info = context.getPackageManager().getPackageInfo(packageName, 0);  
	            userAgent = packageName + "/" + info.versionCode;  
	        } catch (NameNotFoundException e) {  
	        }  
	  
	        if (stack == null) {  
	        //2.3及以上版本使用HurlStack来处理请求  

	        if (Build.VERSION.SDK_INT >= 9) {  
	            stack = new HurlStack();  
	        } else {  
	            // Prior to Gingerbread, HttpUrlConnection was unreliable.  
	            // See: http://android-developers.blogspot.com/2011/09/androids-http-clients.html  
	            stack = new HttpClientStack(AndroidHttpClient.newInstance(userAgent));  
	        }  
	    }  
	  
	    Network network = new BasicNetwork(stack);  
	  
	    RequestQueue queue = new RequestQueue(new DiskBasedCache(cacheDir), network);  
	    queue.start();  
	  
	    return queue;  
	}  

我们来看下HurlStack这个类的构造大家就会发现其实volley可以支持https了，同样位于toolbox包下

	public HurlStack() {  
	       this(null);  
	   }  
	  
	   /** 
	    * @param urlRewriter Rewriter to use for request URLs 
	    */  
	   public HurlStack(UrlRewriter urlRewriter) {  
	       this(urlRewriter, null);  
	   }  
	  
	   /** 
	    * @param urlRewriter Rewriter to use for request URLs 
	    * @param sslSocketFactory SSL factory to use for HTTPS connections 
	    */  
	   public HurlStack(UrlRewriter urlRewriter, SSLSocketFactory sslSocketFactory) {  
	       mUrlRewriter = urlRewriter;  
	       mSslSocketFactory = sslSocketFactory;  
	   }  
	
	private HttpURLConnection openConnection(URL url, Request<?> request) throws IOException {  
	       HttpURLConnection connection = createConnection(url);  
	  
	       int timeoutMs = request.getTimeoutMs();  
	       connection.setConnectTimeout(timeoutMs);  
	       connection.setReadTimeout(timeoutMs);  
	       connection.setUseCaches(false);  
	       connection.setDoInput(true);  
	  
	      <span style="color:#ff6600;"> </span>// use caller-provided custom SslSocketFactory, if any, for HTTPS  
	       if ("https".equals(url.getProtocol()) && mSslSocketFactory != null) {  
	           ((HttpsURLConnection)connection).setSSLSocketFactory(mSslSocketFactory);  
	       }  
	  
	       return connection;  
	   }  

由此可以看出HurlStack 是支持HTTPS 只是在Volley.java生成对象时调用的是无参构造。所以 SSLSocketFactory并没有实例对象。

**那么一种修改的方法是重写Volley.java newRequestQueueInDisk方法 调用第三个构造。又因为这三个构造最后调用的都是参数最多的那个所以也可以在第三个构造中直接默认生成SSLSocketFactory示例。但是我没有用这种方法。**

**我的实现方法是在toolbox中添加HTTPSTrustManager类(代码网上找的- -、)，并对HurlStack的createConnetcion方法进行了小小的修改。**

	package com.android.volley.toolbox;  
	  
	import java.security.KeyManagementException;  
	import java.security.NoSuchAlgorithmException;  
	import java.security.SecureRandom;  
	import java.security.cert.X509Certificate;  
	  
	import javax.net.ssl.HostnameVerifier;  
	import javax.net.ssl.HttpsURLConnection;  
	import javax.net.ssl.SSLContext;  
	import javax.net.ssl.SSLSession;  
	import javax.net.ssl.TrustManager;  
	import javax.net.ssl.X509TrustManager;  
	  
	public class HTTPSTrustManager implements X509TrustManager {  
	  
	    private static TrustManager[] trustManagers;  
	    private static final X509Certificate[] _AcceptedIssuers = new X509Certificate[] {};  
	  
	    @Override  
	    public void checkClientTrusted(  
	            java.security.cert.X509Certificate[] x509Certificates, String s)  
	            throws java.security.cert.CertificateException {  
	        // To change body of implemented methods use File | Settings | File  
	        // Templates.  
	    }  
	  
	    @Override  
	    public void checkServerTrusted(  
	            java.security.cert.X509Certificate[] x509Certificates, String s)  
	            throws java.security.cert.CertificateException {  
	        // To change body of implemented methods use File | Settings | File  
	        // Templates.  
	    }  
	  
	    public boolean isClientTrusted(X509Certificate[] chain) {  
	        return true;  
	    }  
	  
	    public boolean isServerTrusted(X509Certificate[] chain) {  
	        return true;  
	    }  
	  
	    @Override  
	    public X509Certificate[] getAcceptedIssuers() {  
	        return _AcceptedIssuers;  
	    }  
	  
	    public static void allowAllSSL() {  
	        HttpsURLConnection.setDefaultHostnameVerifier(new HostnameVerifier() {  
	  
	            @Override  
	            public boolean verify(String arg0, SSLSession arg1) {  
	                // TODO Auto-generated method stub  
	                return true;  
	            }  
	  
	        });  
	  
	        SSLContext context = null;  
	        if (trustManagers == null) {  
	            trustManagers = new TrustManager[] { new HTTPSTrustManager() };  
	        }  
	  
	        try {  
	            context = SSLContext.getInstance("TLS");  
	            context.init(null, trustManagers, new SecureRandom());  
	        } catch (NoSuchAlgorithmException e) {  
	            e.printStackTrace();  
	        } catch (KeyManagementException e) {  
	            e.printStackTrace();  
	        }  
	  
	        HttpsURLConnection.setDefaultSSLSocketFactory(context  
	                .getSocketFactory());  
	    }  
	  
	}  

createConnction方法的修改

	protected HttpURLConnection createConnection(URL url) throws IOException {  
		    //如果请求是https请求那么就信任所有SSL  
		    if (url.toString().contains("https")) {  
		             HTTPSTrustManager.allowAllSSL();  
		        }  
		        return (HttpURLConnection).url.openConnection();  
	}  

其实就是添加了一个 HTTPSTrustManager类 并在createConnection中调用一下HTTPSTrustManager.allowAllSSL()。

就这么简单= =、
