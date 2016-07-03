---
layout: post
title:  "Retrofit2 OAuth"
date:   2016-06-21 21:51:36 +0200
categories: jekyll update
---
```java
public class ServiceGenerator {
    private static final String API_BASE_URL = "https://rest.sensation.io/";

    private static OkHttpClient.Builder httpClient = new OkHttpClient.Builder();

    private static HttpLoggingInterceptor loggingInterceptor = new HttpLoggingInterceptor();

    private static Retrofit.Builder builder =
            new Retrofit.Builder()
                    .baseUrl(API_BASE_URL)
                    .addConverterFactory(GsonConverterFactory.create());
    public static <S> S createService(Class<S> serviceClass, String apiKey, String apiSecret) {
       if (apiKey != null && apiSecret != null) {
           final ByteString credentialsByteString = ByteString.encodeUtf8(apiKey + ":" + apiSecret);

           loggingInterceptor.setLevel(BuildConfig.DEBUG ?
                   HttpLoggingInterceptor.Level.BODY : HttpLoggingInterceptor.Level.NONE);

           httpClient.addInterceptor(loggingInterceptor)
                   .addInterceptor(new Interceptor() {
                       @Override
                       public Response intercept(Chain chain) throws IOException {
                           Request request = chain. request();
                           Request.Builder requestBuilder = request.newBuilder()
                                   .addHeader("authorization",
                                           "Basic " + credentialsByteString.base64());
                           return chain.proceed(requestBuilder.build());
                       }
                   });
       }
       OkHttpClient client = httpClient.build();
       Retrofit retrofit = builder.client(client).build();
       return retrofit.create(serviceClass);
   }
}
```

```java
private void getAccessToken(final Callback<AccessToken> accessTokenCallback) {
    SensationService service =
            ServiceGenerator.createService(SensationService.class, apiKey, apiSecret);
    Call<AccessToken> call = service.getAccessToken("client_credentials");

    call.enqueue(new Callback<AccessToken>() {

        @Override
        public void onResponse(Call<AccessToken> call, retrofit2.Response<AccessToken> response) {
            if (response.isSuccessful()) {
                accessTokenCallback.onResponse(call, response);
            }
        }

        @Override
        public void onFailure(Call<AccessToken> call, Throwable t) {

        }
    });
}
```
