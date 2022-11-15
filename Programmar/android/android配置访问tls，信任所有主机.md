只需要配置客户端即可

```java
/** okHttp3客户端 */
    public static OkHttpClient client;

    static {
        X509TrustManager x509TrustManager = new X509TrustManager() {
            @SuppressLint("TrustAllX509TrustManager")
            @Override
            public void checkClientTrusted(X509Certificate[] chain, String authType) {
            }

            @SuppressLint("TrustAllX509TrustManager")
            @Override
            public void checkServerTrusted(X509Certificate[] chain, String authType) {
            }

            @Override
            public X509Certificate[] getAcceptedIssuers() {
                return new X509Certificate[]{};
            }
        };
        TrustManager[] trustAllCerts = new TrustManager[]{x509TrustManager};
        try {
            SSLContext sslContext = SSLContext.getInstance("TLS");
            sslContext.init(null, trustAllCerts, new SecureRandom());

            client = new OkHttpClient.Builder()
                .readTimeout(READ_TIMEOUT, TimeUnit.SECONDS)        // 设置读取超时时间
                .writeTimeout(WRITE_TIMEOUT, TimeUnit.SECONDS)      // 设置写的超时时间
                .connectTimeout(CONNECT_TIMEOUT, TimeUnit.SECONDS)  // 设置连接超时时间
                .hostnameVerifier((hostname, session) -> true)      // 信任所有主机
                .sslSocketFactory(sslContext.getSocketFactory(), x509TrustManager)
                .build();
        } catch (NoSuchAlgorithmException | KeyManagementException e) {
            Log.w("ssl get instance", String.valueOf(e));
        }
    }
```