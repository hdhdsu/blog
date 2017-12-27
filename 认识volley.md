## 认识volley
#### 使用volley
1.新建requeseQueue对象（Volley.newRequestQueue(context)）
2.新建request
3.requestQueue.add(request)

#### 解析
1.创建requestQueue

    public static RequestQueue newRequestQueue(Context context, HttpStack stack) {  
    File cacheDir = new File(context.getCacheDir(), DEFAULT_CACHE_DIR);  
    String userAgent = "volley/0";  
    try {  
        String packageName = context.getPackageName();  
        PackageInfo info = context.getPackageManager().getPackageInfo(packageName, 0);  
        userAgent = packageName + "/" + info.versionCode;  
    } catch (NameNotFoundException e) {  
    }  
    if (stack == null) {  
        if (Build.VERSION.SDK_INT >= 9) {  
            stack = new HurlStack();  
        } else {  
            stack = new HttpClientStack(AndroidHttpClient.newInstance(userAgent));  
        }  
    }  
    Network network = new BasicNetwork(stack);  
    RequestQueue queue = new RequestQueue(new DiskBasedCache(cacheDir), network);  
    queue.start();  
    return queue;  
    }  
*  创建了HttpStack（手机版本号>9就使用HttpClient，否则是HttpUrlConnection）
*  创建了NetWork对象（根据HttpStack来处理网络请求的）
*  启动RequestQueue.start()

2.RequestQueue.start()

    public void start() {  
    stop();  // Make sure any currently running dispatchers are stopped.  
    // Create the cache dispatcher and start it.  
    mCacheDispatcher = new CacheDispatcher(mCacheQueue, mNetworkQueue, mCache, mDelivery);  
    mCacheDispatcher.start();  
    // Create network dispatchers (and corresponding threads) up to the pool size.  
    for (int i = 0; i < mDispatchers.length; i++) {  
        NetworkDispatcher networkDispatcher = new NetworkDispatcher(mNetworkQueue, mNetwork,  
                mCache, mDelivery);  
        mDispatchers[i] = networkDispatcher;  
        networkDispatcher.start();  
    }  
    }  
*  创建CacheDisPatcher (缓存线程)和NetWorkDisPatcher（网络线程）默认有五个线程在后台运行，四个参数mNetworkQueue, mNetwork,  mCache, mDelivery，最后一个参数用来对服务器返回的结果进行分发

3.requestQueue.add(request)

        public <T> Request<T> add(Request<T> request) {  
        // Tag the request as belonging to this queue and add it to the set of current requests.  
        request.setRequestQueue(this);  
        synchronized (mCurrentRequests) {  
        mCurrentRequests.add(request);  
        }  
        // Process requests in the order they are added.  
        request.setSequence(getSequenceNumber());  
        request.addMarker("add-to-queue");  
        // If the request is uncacheable, skip the cache queue and go     straight to the network.  
        if (!request.shouldCache()) {  
        mNetworkQueue.add(request);  
        return request;  
        }  
        // Insert request into stage if there's already a request with the     same cache key in flight.  
        synchronized (mWaitingRequests) {  
        String cacheKey = request.getCacheKey();  
        if (mWaitingRequests.containsKey(cacheKey)) {  
            // There is already a request in flight. Queue up.  
            Queue<Request<?>> stagedRequests = mWaitingRequests.get(cacheKey);  
            if (stagedRequests == null) {  
                stagedRequests = new LinkedList<Request<?>>();  
            }  
            stagedRequests.add(request);  
            mWaitingRequests.put(cacheKey, stagedRequests);  
            if (VolleyLog.DEBUG) {  
                VolleyLog.v("Request for cacheKey=%s is in flight, putting on hold.", cacheKey);  
            }  
        } else {  
            // Insert 'null' queue for this cacheKey, indicating there is now a request in  
            // flight.  
            mWaitingRequests.put(cacheKey, null);  
            mCacheQueue.add(request);  
        }  
        return request;  
        }  
        }  
*  分队列，一个缓存队列一个网络队列。
*  cacheDispatcher
如果有缓存并且不过期就直接使用缓存中的数据，直接解析并且回调数据即可。但是如果不符合条件就进行网络请求。
*  networkDispatcher请求数据

        public class NetworkDispatcher extends Thread {  
        ……  
        @Override  
        public void run() {  
        Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);  
        Request<?> request;  
        while (true) {  
            try {  
                // Take a request from the queue.  
                request = mQueue.take();  
            } catch (InterruptedException e) {  
                // We may have been interrupted because it was time to quit.  
                if (mQuit) {  
                    return;  
                }  
                continue;  
            }  
            try {  
                request.addMarker("network-queue-take");  
                // If the request was cancelled already, do not perform the  
                // network request.  
                if (request.isCanceled()) {  
                    request.finish("network-discard-cancelled");  
                    continue;  
                }  
                addTrafficStatsTag(request);  
                // Perform the network request.  
                NetworkResponse networkResponse = mNetwork.performRequest(request);  
                request.addMarker("network-http-complete");  
                // If the server returned 304 AND we delivered a response already,  
                // we're done -- don't deliver a second identical response.  
                if (networkResponse.notModified && request.hasHadResponseDelivered()) {  
                    request.finish("not-modified");  
                    continue;  
                }  
                // Parse the response here on the worker thread.  
                Response<?> response = request.parseNetworkResponse(networkResponse);  
                request.addMarker("network-parse-complete");  
                // Write to cache if applicable.  
                // TODO: Only update cache metadata instead of entire record for 304s.  
                if (request.shouldCache() && response.cacheEntry != null) {  
                    mCache.put(request.getCacheKey(), response.cacheEntry);  
                    request.addMarker("network-cache-written");  
                }  
                // Post the response back.  
                request.markDelivered();  
                mDelivery.postResponse(request, response);  
            } catch (VolleyError volleyError) {  
                parseAndDeliverNetworkError(request, volleyError);  
            } catch (Exception e) {  
                VolleyLog.e(e, "Unhandled exception %s", e.toString());  
                mDelivery.postError(request, new VolleyError(e));  
            }  
        }  
        }  
        }  
调用Network的performRequest()方法来去发送网络请求


     public NetworkResponse performRequest(Request<?> request) throws VolleyError {  
        long requestStart = SystemClock.elapsedRealtime();  
        while (true) {  
            HttpResponse httpResponse = null;  
            byte[] responseContents = null;  
            Map<String, String> responseHeaders = new HashMap<String, String>();  
            try {  
                // Gather headers.  
                Map<String, String> headers = new HashMap<String, String>();  
                addCacheHeaders(headers, request.getCacheEntry());  
                httpResponse = mHttpStack.performRequest(request, headers);  
                StatusLine statusLine = httpResponse.getStatusLine();  
                int statusCode = statusLine.getStatusCode();  
                responseHeaders = convertHeaders(httpResponse.getAllHeaders());  
                // Handle cache validation.  
                if (statusCode == HttpStatus.SC_NOT_MODIFIED) {  
                    return new NetworkResponse(HttpStatus.SC_NOT_MODIFIED,  
                            request.getCacheEntry() == null ? null : request.getCacheEntry().data,  
                            responseHeaders, true);  
                }  
                // Some responses such as 204s do not have content.  We must check.  
                if (httpResponse.getEntity() != null) {  
                  responseContents = entityToBytes(httpResponse.getEntity());  
                } else {  
                  // Add 0 byte response as a way of honestly representing a  
                  // no-content request.  
                  responseContents = new byte[0];  
                }  
                // if the request is slow, log it.  
                long requestLifetime = SystemClock.elapsedRealtime() - requestStart;  
                logSlowRequests(requestLifetime, request, responseContents, statusLine);  
                if (statusCode < 200 || statusCode > 299) {  
                    throw new IOException();  
                }  
                return new NetworkResponse(statusCode, responseContents, responseHeaders, false);  
            } catch (Exception e) {  
                ……  
            }  
        }  
        }  

之后会将服务器返回的数据组装成一个NetworkResponse对象进行返回。在NetworkDispatcher中收到了NetworkResponse这个返回值后又会调用Request的parseNetworkResponse()方法来解析NetworkResponse中的数据，以及将数据写入到缓存，这个方法的实现是交给Request的子类来完成的，因为不同种类的Request解析的方式也肯定不同。

在解析完了NetworkResponse中的数据之后，又会调用ExecutorDelivery的postResponse()方法来回调解析出的数据
每一条网络请求的响应都会调用Request的deliverResponse()方法，每一条网络请求的响应都是回调到这个方法中，最后我们再在这个方法中将响应的数据回调到Response.Listener的onResponse()方法中就可以了。