# Generic-Retrofit-Implementation (Android Kotlin)
Generic code to call api using retrofit2

<b>NOW CALL ANY API BY JUST GIVING THE MODEL CLASS</b>
```
// Create ApiEndpoint and UseCase instances
        val repository = Repository(UseCase<YourModel>())

        CoroutineScope(Dispatchers.Main).launch {
            RetrofitClient.apiEndpoint?.getXyz(
                YourModel(value)
            )?.let {
                repository.fetchData(
                    it
                ) { contactData, error ->
                    if (error != null) {
                        // Handle error
                        Log.d("CheckingResponse", "$error")
                    } else {
                        // ..success
                    }
                  }
                }
        }
```

---------------------------------------------------------------------------------

<i>ADD THIS COMMON CLASS IN YOUR PROJECT DIRECTORY</i>

Dependencies 
```
    //API Calling
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    implementation 'com.squareup.retrofit2:converter-gson:2.9.0'
    implementation 'com.squareup.okhttp3:okhttp:4.9.1'

    //get curl
    implementation 'com.github.grapesnberries:curlloggerinterceptor:0.1'
```

<b> Create RetrofitClient </b>

```
object RetrofitClient {
    private const val BASE_URL = "https://xyz/"
    private var client = OkHttpClient.Builder() // add our curl logger here
        .addInterceptor(CurlLoggerInterceptor()) // curl logger to get curl when any api hits (You need to add dependency to use this)
    private val retrofit: Retrofit = Retrofit.Builder()
        .baseUrl(BASE_URL)
        .addConverterFactory(GsonConverterFactory.create())
        .client(client.build())
        .build()

    val apiEndpoint: ApiEndpoint? = retrofit.create(ApiEndpoint::class.java)

}
```

<b>ApiEndpoint Interface</b>

```
interface ApiEndpoint {
    @POST("path")
    fun getXyz(@Body body:Body): Call<ApiResponse<Model>>
}
```

<b>ApiRespose Data class</b>
```
data class ApiResponse<T>(
    val status: Int,
    val message: String,
    val success: Boolean,
    val data: T
)
```

<b>Repository </b>

```
class Repository<T>(private val useCase: UseCase<T>) {
    fun fetchData(endpoint: Call<ApiResponse<T>>, callback: (T?, String?) -> Unit) {
        useCase.fetchData(endpoint, callback)
    }
}
```

<b>Usecases </b>
```
class UseCase<T> {
    fun fetchData(endpoint: Call<ApiResponse<T>>, callback: (T?, String?) -> Unit) {
        endpoint.enqueue(object : Callback<ApiResponse<T>> {
            override fun onResponse(call: Call<ApiResponse<T>>, response: Response<ApiResponse<T>>) {
                if (response.isSuccessful) {
                    val responseData = response.body()?.data
                    callback(responseData, null)
                } else {
                    Log.d("CheckingResponse", "fetchData: ${response.message()}")
                    callback(null, "Error: ${response.message()}")
                }
            }

            override fun onFailure(call: Call<ApiResponse<T>>, t: Throwable) {
                callback(null, "Network error: ${t.localizedMessage}")
            }
        })
    }
}
```



