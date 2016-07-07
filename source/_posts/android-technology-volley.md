title: Android-Technology-Volley实现网络层
description: Android开发技术：通过Volley框架实现网络层
date: 2015/08/21 13:12:12 
categories: Android
tags: [Android,Technology,Volley,Network]
---
通过调用封装Volley框架，实现应用通信网络层。

<!--more-->

# Volley简介 #
我们平时在开发Android应用的时候不可避免地都需要用到网络技术，而多数情况下应用程序都会使用HTTP协议来发送和接收网络数据。Android系统中主要提供了两种方式来进行HTTP通信，HttpURLConnection和HttpClient，几乎在任何项目的代码中我们都能看到这两个类的身影，使用率非常高。

不过HttpURLConnection和HttpClient的用法还是稍微有些复杂的，如果不进行适当封装的话，很容易就会写出不少重复代码。于是乎，一些Android网络通信框架也就应运而生，比如说AsyncHttpClient，它把HTTP所有的通信细节全部封装在了内部，我们只需要简单调用几行代码就可以完成通信操作了。再比如Universal-Image-Loader，它使得在界面上显示网络图片的操作变得极度简单，开发者不用关心如何从网络上获取图片，也不用关心开启线程、回收图片资源等细节，Universal-Image-Loader已经把一切都做好了。
Android开发团队也是意识到了有必要将HTTP的通信操作再进行简单化，于是在2013年Google I/O大会上推出了一个新的网络通信框架——Volley。Volley可是说是把AsyncHttpClient和Universal-Image-Loader的优点集于了一身，既可以像AsyncHttpClient一样非常简单地进行HTTP通信，也可以像Universal-Image-Loader一样轻松加载网络上的图片。除了简单易用之外，Volley在性能方面也进行了大幅度的调整，它的设计目标就是非常适合去进行数据量不大，但通信频繁的网络操作，而对于大数据量的网络操作，比如说下载文件等，Volley的表现就会非常糟糕。

# Volley实现网络层框架 #
Volley实现网络层框架，代码结构如下：

![](http://7xrjck.com1.z0.glb.clouddn.com/QQ20160707-1.png-blog)

base路径下，是各种网络请求的基础类，VolleyManager.java是网络请求的管理类。具体讲解详见代码注释。

## VolleyManager ##

	package com.wmgj.amen.network;

	import android.content.Context;

	import com.android.volley.Request;
	import com.android.volley.RequestQueue;
	import com.android.volley.toolbox.Volley;
	import com.wmgj.amen.util.LogUtil;

	/**
	 * author : liuyang
	 * date : 2015/8/21
	 * introduce : Volley管理类
	 */
	public class VolleyManager {
	    private static VolleyManager instance;
	    private Context context;
	    private RequestQueue requestQueue;

	    private VolleyManager(Context context) {
	        this.context = context;
	    }

	    //单例模式：同步锁，保证线程安全，双重检查，避免同步锁引起的性能问题
	    public static VolleyManager getInstance(Context context) {
	        if (instance == null) {
	            synchronized (VolleyManager.class) {
	                if (instance == null) {
	                    instance = new VolleyManager(context);
	                }
	            }
	        }
	        return instance;
	    }

	    public <T> void add2RequestQueue(Request<T> request) {
	        if (request == null) {
	            LogUtil.e("request is null", null);
	            return;
	        }
	        getRequestQueue().add(request);
	    }

	    public void cancelPendingRequests(String tag) {
	        //TODO L.jinzhu 应用stop\crash状况下，清空网络请求
	        getRequestQueue().cancelAll(tag);
	    }

	    public RequestQueue getRequestQueue() {
	        if (requestQueue == null) {
	            requestQueue = Volley.newRequestQueue(context.getApplicationContext());
	        }
	        return requestQueue;
	    }
	}

## BaseVolleyGetRequest ##

	package com.wmgj.amen.network.request.base;

	import com.android.volley.Request;

	/**
	 * author : L.jinzhu
	 * date : 2015/8/21
	 * introduce : Volley请求顶级对象
	 */
	public abstract class BaseVolleyGetRequest<REQUEST, RESPONSE> extends BaseVolleyRequest<REQUEST, RESPONSE> {

	    @Override
	    protected int requestType() {
	        return Request.Method.GET;
	    }

	}


## BaseVolleyPostRequest ##
	package com.wmgj.amen.network.request.base;

	import com.android.volley.Request;
	import com.wmgj.amen.util.AppUtils;
	import com.wmgj.amen.util.LogUtil;
	import com.wmgj.amen.util.StringUtils;

	import java.io.ByteArrayOutputStream;
	import java.io.DataOutputStream;
	import java.io.IOException;

	/**
	 * author : L.jinzhu
	 * date : 2015/8/21
	 * introduce : Volley请求顶级对象
	 */
	@SuppressWarnings("unchecked")
	public abstract class BaseVolleyPostRequest<REQUEST, RESPONSE> extends BaseVolleyRequest<REQUEST, RESPONSE> {

	    @Override
	    protected int requestType() {
	        return Request.Method.POST;
	    }

	    /**
	     * 拼接请求前缀
	     *
	     * @param jsonStr 请求json数据
	     */
	    public byte[] addRequestPrefixData(String jsonStr) {
	        ByteArrayOutputStream os = null;
	        DataOutputStream dos = null;
	        byte[] bytes = new byte[]{};
	        try {
	            os = new ByteArrayOutputStream();
	            dos = new DataOutputStream(os);
	            //二进制前缀
	            String userId = AppUtils.getInstance().getUserId();
	            long uid = StringUtils.isNotBlank(userId) ? Long.parseLong(userId) : 0;
	            int version = 100;
	            long tmp = 0;
	            dos.writeLong(uid);//写入uid
	            dos.writeInt(version);//协议版本
	            dos.writeLong(tmp);//占位符
	            dos.write(jsonStr.getBytes("utf-8"));
	            bytes = os.toByteArray();
	        } catch (Throwable e) {
	            LogUtil.e("request add prefix data error", e);
	        } finally {
	            try {
	                if (null != os)
	                    os.close();
	                if (null != dos)
	                    dos.close();
	            } catch (IOException e) {
	                LogUtil.e("request add prefix data error: close io error", e);
	            }
	        }
	        return bytes;
	    }
	}

## BaseVolleyRequest ##
	package com.wmgj.amen.network.request.base;

	import com.android.volley.Response;
	import com.android.volley.VolleyError;

	/**
	 * author : L.jinzhu
	 * date : 2015/8/21
	 * introduce : Volley请求顶级对象
	 */
	public abstract class BaseVolleyRequest<REQUEST, RESPONSE> {

	    protected Response.Listener responseSuccessListener = new Response.Listener() {
	        @Override
	        public void onResponse(Object response) {
	            responseSuccess((RESPONSE) response);
	        }
	    };

	    protected Response.ErrorListener responseErrorListener = new Response.ErrorListener() {
	        @Override
	        public void onErrorResponse(VolleyError error) {
	            responseError(error);
	        }
	    };

	    public abstract REQUEST getRequest();

	    protected abstract String getUrl();

	    protected abstract int requestType();

	    protected abstract String requestTag();

	    /**
	     * 正常响应处理逻辑[需要子类重写]
	     */
	    protected abstract void responseSuccess(RESPONSE response);

	    /**
	     * 异常响应处理逻辑[需要子类重写]
	     */
	    protected abstract void responseError(VolleyError error);
	}

## GetStringRequest ##
	package com.wmgj.amen.network.request.base;

	import com.android.volley.toolbox.StringRequest;

	/**
	 * author : L.jinzhu
	 * date : 2015/8/21
	 * introduce : Volley请求string父对象
	 */
	public abstract class GetStringRequest extends BaseVolleyGetRequest<StringRequest, String> {

	    @SuppressWarnings("unchecked")
	    @Override
	    public StringRequest getRequest() {
	        return new StringRequest(requestType(), getUrl(),
	                responseSuccessListener, responseErrorListener);
	    }

	}

## PostJsonRequest ##
	package com.wmgj.amen.network.request.base;

	import android.content.Context;
	import android.os.Bundle;
	import android.os.Handler;
	import android.os.Message;

	import com.android.volley.DefaultRetryPolicy;
	import com.android.volley.NetworkResponse;
	import com.android.volley.ParseError;
	import com.android.volley.Response;
	import com.android.volley.VolleyError;
	import com.android.volley.toolbox.HttpHeaderParser;
	import com.android.volley.toolbox.JsonObjectRequest;
	import com.android.volley.toolbox.JsonRequest;
	import com.wmgj.amen.common.MessageSignConstant;
	import com.wmgj.amen.util.LogUtil;

	import org.json.JSONObject;

	import java.io.UnsupportedEncodingException;

	/**
	 * author : L.jinzhu
	 * date : 2015/8/21
	 * introduce : Volley请求json父对象
	 */
	public abstract class PostJsonRequest extends BaseVolleyPostRequest<JsonRequest, JSONObject> {
	    // 操作超时时间
	    final int CUD_SOCKET_TIMEOUT = 10000;
	    // 最大重试请求次数
	    final int MAX_RETRIES = 0;

	    public Handler handler;
	    public Context context;

	    /**
	     * 请求参数json[需要子类重写]
	     */
	    protected abstract String getParamsJson();

	    @SuppressWarnings("unchecked")
	    @Override
	    public JsonRequest getRequest() {
	        JsonObjectRequest request = null;
	        try {
	            request = new JsonObjectRequest(requestType(), getUrl(), null,
	                    responseSuccessListener, responseErrorListener) {
	                @Override
	                public byte[] getBody() {
	                    return addRequestPrefixData(getParamsJson());
	                }

	                @Override
	                protected Response<JSONObject> parseNetworkResponse(NetworkResponse response) {
	                    //转换编码，解决中文乱码问题
	                    try {
	                        JSONObject jsonObject = new JSONObject(new String(response.data, "UTF-8"));
	                        return Response.success(jsonObject, HttpHeaderParser.parseCacheHeaders(response));
	                    } catch (UnsupportedEncodingException e) {
	                        return Response.error(new ParseError(e));
	                    } catch (Exception je) {
	                        return Response.error(new ParseError(je));
	                    }
	                }
	            };
	            // 超时时间、重试次数设定
	            request.setRetryPolicy(new DefaultRetryPolicy(CUD_SOCKET_TIMEOUT, MAX_RETRIES, DefaultRetryPolicy.DEFAULT_BACKOFF_MULT));

	            if (requestTag() != null && !"".equals(requestTag()))
	                request.setTag(requestTag());
	            LogUtil.i("request json: [" + requestTag() + "]: " + new String(request.getBody()).substring(20));
	        } catch (Throwable e) {
	            LogUtil.e("request json error [" + requestTag() + "] ", e);
	        }
	        return request;
	    }

	    @Override
	    protected void responseError(VolleyError error) {
	        LogUtil.e("response error json: [" + requestTag() + "]: ", error.getCause());
	        if (null == handler)
	            return;
	        Bundle b = new Bundle();
	        Message msg = new Message();
	        b.putString("message", VolleyErrorHelper.getMessage(error, context));
	        msg.what = MessageSignConstant.SERVER_OR_NETWORK_ERROR;
	        msg.setData(b);
	        handler.sendMessage(msg);
	    }
	}

## PostStringRequest ##
	package com.wmgj.amen.network.request.base;

	import com.android.volley.NetworkResponse;
	import com.android.volley.ParseError;
	import com.android.volley.Response;
	import com.android.volley.toolbox.HttpHeaderParser;
	import com.android.volley.toolbox.StringRequest;
	import com.wmgj.amen.util.LogUtil;

	import java.io.UnsupportedEncodingException;

	/**
	 * author : L.jinzhu
	 * date : 2015/8/21
	 * introduce : Volley请求string父对象
	 */
	public abstract class PostStringRequest extends BaseVolleyPostRequest<StringRequest, String> {
	    /**
	     * 请求参数json[需要子类重写]
	     */
	    protected abstract String getParamsJson();

	    @SuppressWarnings("unchecked")
	    @Override
	    public StringRequest getRequest() {
	        StringRequest request = null;
	        try {
	            request = new StringRequest(requestType(), getUrl(),
	                    responseSuccessListener, responseErrorListener) {
	                @Override
	                public byte[] getBody() {
	                    return addRequestPrefixData(getParamsJson());
	                }

	                @Override
	                protected Response<String> parseNetworkResponse(NetworkResponse response) {
	                    //转换编码，解决中文乱码问题
	                    try {
	                        String object = new String(response.data, "UTF-8");
	                        return Response.success(object, HttpHeaderParser.parseCacheHeaders(response));
	                    } catch (UnsupportedEncodingException e) {
	                        return Response.error(new ParseError(e));
	                    } catch (Exception je) {
	                        return Response.error(new ParseError(je));
	                    }
	                }
	            };
	            if (requestTag() != null && "".equals(requestTag()))
	                request.setTag(requestTag());
	            LogUtil.i("request json: " + new String(request.getBody()).substring(20));
	        } catch (Throwable e) {
	            LogUtil.e("request json error", e);
	        }
	        return request;
	    }
	}

## VolleyErrorHelper ##
	package com.wmgj.amen.network.request.base;

	import android.content.Context;

	import com.android.volley.AuthFailureError;
	import com.android.volley.NetworkError;
	import com.android.volley.NetworkResponse;
	import com.android.volley.NoConnectionError;
	import com.android.volley.ServerError;
	import com.android.volley.TimeoutError;
	import com.android.volley.VolleyError;
	import com.google.gson.Gson;
	import com.google.gson.reflect.TypeToken;
	import com.wmgj.amen.R;

	import java.util.HashMap;
	import java.util.Map;

	/**
	 * author : L.jinzhu
	 * date : 2015/8/21
	 * introduce : Volley请求错误码
	 */
	public class VolleyErrorHelper {
	    /**
	     * 获取异常信息
	     */
	    public static String getMessage(Object error, Context context) {
	        if (error instanceof TimeoutError) {
	            return context.getResources().getString(R.string.network_timeout_error);
	        } else if (isServerProblem(error)) {
	            return context.getResources().getString(R.string.server_error);
	        } else if (isNetworkProblem(error)) {
	            return context.getResources().getString(R.string.network_error);
	        }
	        return context.getResources().getString(R.string.server_error);
	    }

	    /**
	     * 网络错误
	     */
	    private static boolean isNetworkProblem(Object error) {
	        return (error instanceof NetworkError) || (error instanceof NoConnectionError);
	    }

	    /**
	     * 服务器错误
	     */
	    private static boolean isServerProblem(Object error) {
	        return (error instanceof ServerError) || (error instanceof AuthFailureError);
	    }

	    /**
	     * 服务器错误
	     */
	    private static String handleServerError(Object err, Context context) {
	        VolleyError error = (VolleyError) err;
	        NetworkResponse response = error.networkResponse;
	        if (response != null) {
	            switch (response.statusCode) {
	                case 404:
	                case 422:
	                case 401:
	                    try {
	                        // server might return error like this { "error": "Some error occured" }
	                        // Use "Gson" to parse the result
	                        HashMap<String, String> result = new Gson().fromJson(new String(response.data),
	                                new TypeToken<Map<String, String>>() {
	                                }.getType());

	                        if (result != null && result.containsKey("error")) {
	                            return result.get("error");
	                        }

	                    } catch (Exception e) {
	                        e.printStackTrace();
	                    }
	                    return error.getMessage();
	                default:
	                    return context.getResources().getString(R.string.server_error);
	            }
	        }
	        return context.getResources().getString(R.string.server_error);
	    }
	}

## 业务调用类：FriendAddRspRequest ##
	package com.wmgj.amen.network.request;

	import android.content.Context;
	import android.content.Intent;
	import android.os.Bundle;
	import android.os.Handler;
	import android.os.Message;

	import com.wmgj.amen.common.Actions;
	import com.wmgj.amen.common.MessageSignConstant;
	import com.wmgj.amen.common.ResponseConstant;
	import com.wmgj.amen.common.WebConstant;
	import com.wmgj.amen.dao.Task4FriendDao;
	import com.wmgj.amen.dao.UserDao;
	import com.wmgj.amen.dao.impl.Task4FriendDaoImpl;
	import com.wmgj.amen.dao.impl.UserDaoImpl;
	import com.wmgj.amen.entity.task.Task4Friend;
	import com.wmgj.amen.entity.net.request.FriendActionInfo;
	import com.wmgj.amen.entity.net.request.base.RequestInfo;
	import com.wmgj.amen.entity.net.response.base.ResponseInfo;
	import com.wmgj.amen.entity.user.User;
	import com.wmgj.amen.network.request.base.PostJsonRequest;
	import com.wmgj.amen.util.GsonUtil;
	import com.wmgj.amen.util.LogUtil;
	import com.wmgj.amen.util.UserUtils;

	import org.json.JSONObject;

	/**
	 * author : L.jinzhu
	 * date : 2015/9/11
	 * introduce : 好友添加响应（同意或者拒绝）request
	 */
	public class FriendAddRspRequest extends PostJsonRequest {
	    private User user;
	    private int typeId;

	    public FriendAddRspRequest(Handler h, Context c, User user, int typeId) {
	        this.handler = h;
	        this.context = c;
	        this.user = user;
	        this.typeId = typeId;
	    }

	    @Override
	    protected String getParamsJson() {
	        FriendActionInfo actionInfo = new FriendActionInfo(23, user.getUid(), typeId);
	        RequestInfo r = new RequestInfo(context, actionInfo);
	        return GsonUtil.toJson(r);
	    }

	    @Override
	    protected String getUrl() {
	        return WebConstant.BASE_URL;
	    }

	    @Override
	    protected String requestTag() {
	        return "friendAddRsp";
	    }

	    @Override
	    protected void responseSuccess(JSONObject response) {
	        Bundle b = new Bundle();
	        Message msg = new Message();
	        try {
	            LogUtil.i("response success json: [" + requestTag() + "]: " + response.toString());
	            ResponseInfo info = GsonUtil.fromJson(response.toString(), ResponseInfo.class);
	            //响应正常
	            if (ResponseConstant.SUCCESS == info.getCode()) {
	                //好友保存到数据库
	                UserDao userDao = new UserDaoImpl();
	                userDao.add(user, User.STATUS_FRIEND);
	                //好友对应的任务Task，更新状态到数据库
	                Task4FriendDao task4FriendDao = new Task4FriendDaoImpl();
	                task4FriendDao.updateTask(user.getUid(), Task4Friend.APPLY_STATUS_MY_RESPONSE_AGREE);

	                //广播通知通讯录、用户详情页UI
	                Intent intent = new Intent(Actions.ACTION_FRIEND_IS_FRIEND);
	                intent.putExtra("userId", user.getUid());
	                context.sendBroadcast(intent);
	                // 清空好友列表缓存
	                UserUtils.getInstance().cleanFriendList();

	                msg.what = MessageSignConstant.FRIEND_ADD_RSP_SUCCESS;
	                b.putLong("userId", user.getUid());
	                msg.setData(b);
	                handler.sendMessage(msg);
	                LogUtil.i("friend add rsp success");
	            }
	            //响应失败
	            else {
	                b.putInt("code", info.getCode());
	                b.putString("message", info.getMessage());
	                msg.what = MessageSignConstant.FRIEND_ADD_RSP_FAILURE;
	                msg.setData(b);
	                handler.sendMessage(msg);
	                LogUtil.e("friend add rsp failure: code: " + info.getCode() + ",message: " + info.getMessage(), null);
	            }
	        } catch (Throwable e) {
	            handler.sendEmptyMessage(MessageSignConstant.UNKNOWN_ERROR);
	            LogUtil.e("friend add rsp error", e);
	        }
	    }
	}


