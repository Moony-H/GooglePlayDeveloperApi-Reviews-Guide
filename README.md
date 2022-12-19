# GooglePlayDeveloperApi-Reviews-Guide

이번에는 리뷰를 가져오고 답하는 api에 대해 설명하겠습니다.

<br/>

## 사용 환경

이번 글에서 사용할 언어는 Kotlin 입니다. compiler로는 Intellij CE를 사용합니다.

또한 Retrofit2를 이용하여 Rest API를 호출할 것입니다.

이 글을 보기 전에 앞서 설명한 [Google Play Console API Guide](https://github.com/Moony-H/GooglePlayConsoleAPIGuide)의 내용을 먼저 보시기 바랍니다.

##  reviews list

가장 먼저 볼 api는 list 입니다.

<br/>


list는 여러 리뷰를 가져올 수 있습니다.

모든 리뷰를 주는 api는 아니기 때문에 일부의 review를 보고 다음 리뷰를 보고 싶다면

response로 오는 pagenationToken을 token query parameter에 사용하여 다음 페이지에 있는 리뷰를 볼 수 있습니다.

token parameter에 아무것도 넣지 않는다면 맨 처음 리뷰가 있는 첫 페이지 부터 주어집니다.

<br/>


먼저 아래와 같이 comment 객체를 작성합니다.

<br/>

**Comment.kt**

```kotlin
data class Comment(
    @SerializedName("userComment")
    val userComment: UserComment,
    @SerializedName("developerComment")
    val developerComment: DeveloperComment
) {

    data class UserComment(
        @SerializedName("text")
        val text: String,
        @SerializedName("lastModified")
        val lastModified: Timestamp,
        @SerializedName("starRating")
        val starRating: Int,
        @SerializedName("reviewerLanguage")
        val reviewerLanguage: String,
        @SerializedName("device")
        val device: String,
        @SerializedName("androidOsVersion")
        val androidOsVersion: Int,
        @SerializedName("appVersionCode")
        val appVersionCode: Int,
        @SerializedName("appVersionName")
        val appVersionName: String,
        @SerializedName("thumbsUpCount")
        val thumbsUpCount: Int,
        @SerializedName("thumbsDownCount")
        val thumbsDownCount: Int,
        @SerializedName("deviceMetadata")
        val deviceMetadata: DeviceMetadata,
        @SerializedName("originalText")
        val originalText: String
    )

    data class DeveloperComment(
        @SerializedName("text")
        val text: String,
        @SerializedName("lastModified")
        val lastModified: Timestamp
    )

    data class Timestamp(
        @SerializedName("seconds")
        val seconds: String,
        @SerializedName("nanos")
        val nanos: Long

    )

    data class DeviceMetadata(
        @SerializedName("productName")
        val productName: String,
        @SerializedName("manufacturer")
        val manufacturer: String,
        @SerializedName("deviceClass")
        val deviceClass: String,
        @SerializedName("screenWidthPx")
        val screenWidthPx: Int,
        @SerializedName("screenHeightPx")
        val screenHeightPx: Int,
        @SerializedName("nativePlatform")
        val nativePlatform: String,
        @SerializedName("screenDensityDpi")
        val screenDensityDpi: Int,
        @SerializedName("glEsVersion")
        val glEsVersion: Int,
        @SerializedName("cpuModel")
        val cpuModel: String,
        @SerializedName("cpuMake")
        val cpuMake: String,
        @SerializedName("ramMb")
        val ramMb: Int
    )

}

```

<br/>

<br/>

그 다음 Review객체를 생성합니다.

<br/>

**Review.kt**
```kotlin

data class Review(
    @SerializedName("reviewId")
    val reviewId: String,
    @SerializedName("authorName")
    val authorName: String,
    @SerializedName("comments")
    val comments: List<Comment>
)

```

<br/>

그 다음 아래와 같이 response 객체를 생성합니다.

<br/>

**GetReviewListResponseData.kt**

```kotlin
data class GetReviewListResponseData(
    @SerializedName("reviews")
    val reviews: ArrayList<Review>,
    @SerializedName("tokenPagination")
    val tokenPagination: TokenPagination,
    @SerializedName("pageInfo")
    val pageInfo:PageInfo
) {
    data class TokenPagination(
        @SerializedName("nextPageToken")
        val nextPageToken: String
    )
    data class PageInfo(
        @SerializedName("totalResults")
        val totalResults:Int,
        @SerializedName("resultPerPage")
        val resultPerPage:Int,
        @SerializedName("startIndex")
        val startIndex:Int
    )
}
```

<br/>


<br/>


마지막으로 ReviewApi interface를 작성합니다.

<br/>

**ReviewApi.kt**

```kotlin
interface ReviewApi {
    @GET("{packageName}/reviews")
    suspend fun getReviewList(
        @Header("Authorization")
        token:String,
        @Path("packageName")
        packageName: String,
        @Query("token")
        paginationToken: String? = null,
        @Query("startIndex")
        startIndex:Int?,
        @Query("maxResults")
        maxResults:Int?,
        @Query("translationLanguage")
        translationLanguage: String? = null
    ): Response<GetReviewsResponseData>
}

```

<br/>

<br/>

그 다음 위의 interface로 retrofit 객체를 만듭니다.


<br/>


**Main.kt**

```kotlin
val reviewRetrofit = Retrofit.Builder()
        .baseUrl("https://www.googleapis.com/androidpublisher/v3/applications/")
        .addConverterFactory(GsonConverterFactory.create())
        .addCallAdapterFactory(CoroutineCallAdapterFactory())
        .client(OkHttpClient.Builder().build())
        .build()
        .create(ReviewApi::class.java)
```

<br/>

<br/>

그 다음 retrofit 객체로 api를 호출합니다.

<br/>

**Main.kt**

```kotlin
    val reviewListResponse = reviewRetrofit.getReviewList("Bearer $token","{Package Name}","{Page Token}", startIndex = null, maxResults = 0)
    println("review list response code: ${reviewListResponse.code()}")
    println("review list response body: ${reviewListResponse.body()}")
```

<br/>

<br/>

## Reviews get

리뷰 하나를 가져올 수 있는 api입니다.

리뷰 하나만 가져오기 때문에 어떤 리뷰인지 식별할 reviewId가 필요 합니다.

이 reviewId를 얻기 위해서는 위의 list를 사용해야 토큰을 알 수 있습니다.

<br/>

<br/>


먼저 ReviewApi.kt에 아래와 같은 코드를 추가합니다.

<br/>

**ReviewApi.kt**

```kotlin
    @GET("{packageName}/reviews/{reviewId}")
    suspend fun getReview(
        @Header("Authorization")
        token: String,
        @Path("packageName")
        packageName: String,
        @Path("reviewId")
        reviewId: String,
        @Query("translationLanguage")
        translationLanguage: String? = null
    ): Response<Review>

```

<br/>

<br/>

그 다음 **Main.kt**애 코드를 추가하여 api를 호출합니다.

<br/>


**Main.kt**

```kotlin

val reviewResponse=reviewRetrofit.getReview("Bearer $token","{packageName}","reviewId")
println("review response code: ${reviewResponse.code()}")
println("review response body: ${reviewResponse.body()}")

```

<br/>

<br/>


## Reviews reply

리뷰에 답을 할 수 있는 api입니다.

위의 get과 마찬가지로, 리뷰를 식별해야 어떤 리뷰에 답할지 선택할 수 있기 때문에

list api가 먼저 선행 되어 reviewId를 저장해야 합니다.

먼저 **PostReplyReviewBody.kt**파일을 작성합니다.

<br/>

```kotlin
data class PostReplyReviewBody(
    @SerializedName("replyText")
    val replyText:String
)
```

<br/>

<br/>

그 다음 response를 받을 **PostReplyReviewResponseData.kt**를 작성합니다.

<br/>

```kotlin
data class PostReplyReviewResponseData(
    val result:Result
){
    data class Result(
        @SerializedName("replyText")
        val replyText:String,
        @SerializedName("lastEdited")
        val lastEdited: LastEdited
    ){
        data class LastEdited(
            @SerializedName("seconds")
            val seconds:String,
            @SerializedName("nanos")
            val nanos:Long
        )
    }
}
```

<br/>

<br/>


마지막으로 아래와 같은 코드를 **ReviewApi.kt**에 추가합니다.

<br/>

```kotlin
    @POST("{packageName}/reviews/{reviewId}:reply")
    suspend fun postReplyReview(
        @Path("packageName")
        packageName: String,
        @Path("reviewId")
        reviewId: String,
        @Query("access_token")
        accessToken: String,
        @Body
        body: PostReplyReviewBody
    ): Response<PostReplyReviewResponseData>
```

<br/>

<br/>

이제 Main.kt에 아래의 코드로 api를 호출합니다.

<br/>

```kotlin
val response=reviewRetrofit.postReplyReview("{packageName}","{reviewId}",token, PostReplyReviewBody("reply"))
println("review reply response code: ${response.code()}")
println("review reply response body: ${response.body()}")
```

<br/>

<br/>

이로서 모든 api의 method를 설명하였습니다.





