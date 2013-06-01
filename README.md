*Android Networking Made Easy*

![](ion-sample/ion-sample.png)

#### Features
 * Asynchronously download:
   * [Images](https://github.com/koush/ion#load-an-image-into-an-imageview) (into ImageViews or Bitmaps)
   * [JSON](https://github.com/koush/ion#get-json)
   * Strings
   * [Files](https://github.com/koush/ion#download-a-file-with-a-progress-bar)
   * Java types using [Gson](https://github.com/koush/ion#seamlessly-use-your-own-java-classes-with-gson)
 * Easy to use Fluent API designed for Android
   * Automatically cancels operations when the calling Activity finishes
   * Manages invocation back onto the UI thread
   * All operations return a [Future](https://github.com/koush/ion#futures) and [can be cancelled](https://github.com/koush/ion#cancelling-requests)
 * HTTP POST/PUT:
   * text/plain
   * [application/json](https://github.com/koush/ion#post-json-and-read-json)
   * [application/x-www-form-urlencoded](https://github.com/koush/ion#post-applicationx-www-form-urlencoded-and-read-a-string)
   * [multipart/form-data](https://github.com/koush/ion#post-multipartform-data-and-read-json)
 * Transparent usage of HTTP features and optimizations:
   * Caching
   * Gzip/Deflate Compression
   * Connection reuse
   * Cookies
 * Request level [logging and profiling](https://github.com/koush/ion#logging)
 * [Download progress callbacks](https://github.com/koush/ion#download-a-file-with-a-progress-bar)
 * Supports file:/, http(s):/, and content:/ URIs
 * Based on [NIO](http://en.wikipedia.org/wiki/New_I/O) and [AndroidAsync](https://github.com/koush/AndroidAsync)

#### Samples

The included documented [ion-sample](https://github.com/koush/ion/tree/master/ion-sample) project includes some samples that demo common Android network operations:

 * [Twitter Client Sample](https://github.com/koush/ion/blob/master/ion-sample/src/com/koushikdutta/ion/sample/Twitter.java)
   * Download JSON from a server (twitter feed)
   * Populate a ListView Adapter and fetch more data as you scroll to the end
   * Put images from a URLs into ImageViews (twitter profile pictures)
 * File Download with [Progress Bar Sample](https://github.com/koush/ion/blob/master/ion-sample/src/com/koushikdutta/ion/sample/ProgressBarDownload.java)
 
#### More Examples

Looking for more? Check out the examples below that demonstrate some other common scenarios. You can also take a look
at 30+ ion unit tests in the [ion-test](https://github.com/koush/ion/tree/master/ion-test/src/com/koushikdutta/ion/test).

#### Get JSON

```java
Ion.with(context, "http://example.com/thing.json")
.asJSONObject()
.setCallback(new FutureCallback<JSONObject>() {
   @Override
    public void onCompleted(Exception e, String result) {
        // do stuff with the result or error
    }
});
```

#### Post JSON and read JSON

```java
JSONObject json = new JSONObject();
json.putString("foo", "bar");

Ion.with(context, "http://example.com/post")
.setJSONObjectBody(json)
.asJSONObject()
.setCallback(new FutureCallback<JSONObject>() {
   @Override
    public void onCompleted(Exception e, String result) {
        // do stuff with the result or error
    }
});
```

#### Post application/x-www-form-urlencoded and read a String

```java
Ion.with(getContext(), "https://koush.clockworkmod.com/test/echo")
.setBodyParameter("goop", "noop")
.setBodyParameter("foo", "bar")
.asString()
.setCallback(...)
```

#### Post multipart/form-data and read JSON

```java
Ion.with(getContext(), "https://koush.clockworkmod.com/test/echo")
.setMultipartParameter("goop", "noop")
.setMultipartFile("filename.zip", new File("/sdcard/filename.zip"))
.asJSONObject()
.setCallback(...)
```

#### Download a File with a progress bar

```java
Ion.with(context, "http://example.com/really-big-file.zip")
.progressBar(progressBar)
// can also use a custom callback
.progress(new ProgressCallback() {@Override
   public void onProgress(int downloaded, int total) {
       System.out.println("" + downloaded + " / " + total);
   }
})
.write(new File("/sdcard/cm-11.zip")
.setCallback(new FutureCallback<File>() {
   @Override
    public void onCompleted(Exception e, File file) {
        // download done...
        // do stuff with the File or error
    }
});
```

#### Setting Headers

```java
Ion.with(context, "http://example.com/test.txt")
// set the header
.setHeader("foo", "bar")
.asString()
.setCallback(...)
```

#### Load an image into an ImageView

```java
// This is the "long" way to do build an ImageView request... it allows you to set headers, etc.
Ion.with(context, "http://example.com/image.png")
.withBitmap()
.placeholder(R.drawable.placeholder_image)
.error(R.drawable.error_image)
.animateLoad(spinAnimation)
.animateIn(fadeInAnimation)
.intoImageView(imageView);

// but for brevity, use the ImageView specific builder...
Ion.with(imageView)
.placeholder(R.drawable.placeholder_image)
.error(R.drawable.error_image)
.animateLoad(spinAnimation)
.animateIn(fadeInAnimation)
.load("http://example.com/image.png");
```

The Ion Image load API has the following features:
 * Disk and memory caching
 * Bitmaps are held via weak references so memory is managed very effeciently
 * ListView Adapter recycling support
 * Bitmap transformations via the .transform(Transform)
 * Animate loading and loaded ImageView states

#### Futures

_All_ operations return a custom [Future](http://developer.android.com/reference/java/util/concurrent/Future.html) that allows
you to specify a callback that runs on completion.

```java
public interface Future<T> extends Cancellable, java.util.concurrent.Future<T> {
    /**
     * Set a callback to be invoked when this Future completes.
     * @param callback
     * @return
     */
    public Future<T> setCallback(FutureCallback<T> callback);
}

Future<String> string = Ion.with(context)
    .load("http://example.com/string.txt")
    .asString();

Future<JSONObject> json = Ion.with(context)
    .load("http://example.com/json.json")
    .asJSONObject();

Future<File> file = Ion.with(context)
    .load("http://example.com/file.zip")
    .write(new File("/sdcard/file.zip"));

Future<Bitmap> bitmap = Ion.with(context)
    .load("http://example.com/image.png")
    .intoImageView(imageView);

```

#### Cancelling Requests

Futures can be cancelled by calling .cancel():

```java
bitmap.cancel();
json.cancel();
```

#### Blocking on Requests

Though you should try to use callbacks for handling requests whenever possible, blocking on requests is possible too.
All Futures have a Future<T>.get() method that waits for the result of the request, by blocking if necessary.

```java
JSONObject json = Ion.with(context, "http://example.com/thing.json").asJSONObject().get();
```


#### Seamlessly use your own Java classes with [Gson](https://code.google.com/p/google-gson/)
```java
public static class Tweet {
    public String id;
    public String text;
    public String photo;
}

public void getTweets() throws Exception {
    Ion.with(context)
    .load("http://example.com/api/tweets")
    .as(new TypeToken<List<Tweet>>(){});
    .setCallback(new FutureCallback<List<Tweet>>() {
       @Override
        public void onCompleted(Exception e, List<Tweet> tweets) {
          // chirp chirp
        }
    });
}
```

#### Logging

Wondering why your app is slow? Ion lets you do both global and request level logging.

To enable it globally:

```java
Ion.getDefault(getContext()).setLogging("MyLogs", Log.DEBUG);
```

Or to enable it on just a single request:

```java
Ion.with(context, "http://example.com/thing.json")
.setLogging("MyLogs", Log.DEBUG)
.asJSONObject();
```

Log entries will look like this:

```
D/MyLogs(23153): (0 ms) http://example.com/thing.json: Executing request.
D/MyLogs(23153): (106 ms) http://example.com/thing.json: Connecting socket
D/MyLogs(23153): (2985 ms) http://example.com/thing.json: Response is not cacheable
D/MyLogs(23153): (3003 ms) http://example.com/thing.json: Connection successful
```
