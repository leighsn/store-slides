![](/Users/Nakhimovich/Desktop/title.png)
# [fit] Store
# [fit] _**Data Loading Made Easy-ish**_
# [fit] **Mike Nakhimovich - NY Times**
---

^NY Times App, Consulting Apps, Hack Day Apps, Apps for Blog Posts and Talks

# [fit] We _**:heart:**_
# [fit] Making Apps

---
#[fit] We _**:heart:**_
#[fit] Open Source
---
# [fit] Why?
# [fit] Open Source Makes Life Easier
---
![](networking.png)
#[fit] Networking is easier:
# ~~_**HTTPURLCONNECTION**_~~ 
## Volley 
## Retrofit / Okhttp
---
![](storage.png)
# Storage is Easier
# ~~_**Shared Preferences**_~~ 
## Firebase
##Realm
##SqlBrite/SqlDelight
---
![](transformation.jpg)
#[fit] Parsing is Easier
#~~_**JSONObject**_~~ 
##Jackson
##Gson
##Moshi
---
![](easy.jpg)
##Fetching, Persisting & Parsing data has become 
# _**easy**_
---
![](noteasy.jpg)
#[fit] What's NOT Easy?
#[fit] _**DATA LOADING**_
#[fit] Everyone does it differently
---
^Now raise your hand if you think the person sitting next to you does it in the same way
#[fit]  What is _**Data Loading**_?
![](computerScreen.jpg)
# The Act of getting data from an _**external system**_ to a _**user's screen**_
---
^We know that we're gonna use these great network libraries but what is actually doing the network call? does my activity really need to make it?  Who are what retains the data? how much do we need to serialize?
# **Loading is complicated** 
#[fit] Rotation Handling is a special 
#**_snowflake_**
![](snowflake.jpg)

---

# New York Times built **Store** to simplify data loading
## github.com/NYTimes/store
![inline filtered](nytstore.jpg)

---
![](super-hero.gif)
# [FIT]**_OUR GOALS_**
---
![fit](rotation.png)
## Data should survive configuration changes
### agnostic of where it comes from or how it is needed
---
^Activities should deal with activity stuff, presenters should deal with presenting. Stores should be used to store data
# Activities and presenters should stop retaining MBs of data

---
#[fit] Offline as configuration
## Caching as standard, 
## not an exception
---
^Our team becomes 50% bigger every summer
#API should be **simple** enough for an intern to use, yet **robust** enough for every data load.
---
![](nytimes.jpg)
# How do we work with Data at NY Times?
---
#[fit] **80% case**:
# Need data, don't care if fresh or cached
---
![](cloud.png)
#[fit] Need fresh data
##**background updates** & 
##**pull to refresh**
---
# [fit] Requests need to be 
#[fit]**async** & **reactive**
---
^Anytime we fetch an article we want comments too
# Data is dependent on each other,
# Data Loading should be too

---
![](inflight.jpg)
# Performance is important
###New calls should hook into in flight responses
---
# **Parsing** should be done efficiently and minimally
#[fit] Parse once and cache the result for future calls
---
# Let's Use Repository Pattern
###By creating reactive and persistent Data Stores
---
#[fit] **Repository Pattern**
### Separate the logic that retrieves the data and maps it to the [view] model from the [view] logic that acts on the model.
### The repository mediates between the data layer and the [view] layer of the application.
---

#Why Repository?
## Maximize the amount of code that can be tested with automation by isolating the data layer
---
# **Why Repository?**
##Makes it easier to have multiple devs working on same feature
---
# **Why Repository?**
## Data source from many locations will be centrally managed with consistent access rules and logic
---

# Our Implementation
## https://github.com/NYTimes/Store
![inline filtered](nytstore.jpg)

—
# What is a **Store?**
^ Stores bring together all those great open source libs from earlier
## A class that manages the fetching, parsing, and storage of a specific data model
---
# [fit]**Tell a Store:**
#  What to **fetch**
---
# [fit]**Tell a Store:**
#  What to **fetch**
#  Where to **cache**
---
# [fit]**Tell a Store:**
#  What to **fetch**
#  Where to **cache**
# How to **parse**
---
# [fit]**Tell a Store:**
#  What to **fetch**
#  Where to **cache**
#  How to **parse**
# [fit] The Store handles  flow

---

#[fit] ![inline](rxjava.png) Stores are Observable ![inline](rxjava.png)

```java
Observable<T> get( V key);

Observable<T> fetch(V key)

Observable<T> stream()

void clear(V key)

```
---
![](achieve.jpg)

#Let's see how stores helped us achieve our goals through a Pizza Example
---
##Get is for Handling our 80% Use Case
```java
public final class PizzaActivity {
   Store<Pizza, String> pizzaStore;
   void onCreate() {
       store.get("cheese")
            .subscribe(pizza ->.show(pizza));
       }
   }
```
---  
# [fit]How does it flow?
 ![inline](/Users/Nakhimovich/Pictures/get.png)

---
#On Configuration Change
## Save/Restore only your  🧀 and get your cached 🍕  
```java
public final class PizzaActivity {
   void onRecreate(String topping) {
       store.get(topping)
            .subscribe(pizza ->.show(pizza));
       }
   }
```
##Please do not  serialize a Pizza
###It would be very messy

---

# What we gain?:
###Fragments/Presenters don't need to be retained
###No TransactionTooLargeExceptions
###Only keys need to be Parcelable
---

##Efficiency is Important:
```java
for(i=0;i<20;i++){
   store.get(topping)
               .subscribe(pizza -> getView()
               .setData(pizza));
}
```
##Many concurrent Get requests will still only hit network once
---


#[fit] Fetching New Data
```java
public class PizzaPresenter {
   Store<Pizza, String> store;

   void onPTR() {
      store.fetch("cheese")
              .subscribe(pizza ->
                getView().setData(pizza));
          }
      }
```
---
# [fit]How does Store route the request?
![fit inline](/Users/Nakhimovich/Pictures/fetch.png)
##Fetch  also throttles multiple requests and broadcast result
---
# [fit] Stream listens for events
```java
public class PizzaBar {
   Store<Pizza, String> store;
   void showPizzaDone() {
       store.stream()
       .subscribe(pizza ->
          getView().showSnackBar(pizza));
  }
}
```

---

![](buildingstore.jpg)
## **How do We build a Store?**
### By implementing interfaces
---
# Interfaces
```java
Fetcher<Raw, Key>{
   Observable<Raw> fetch(Key key);
}
Persister<Raw, Key>{}
   Observable<Raw> read(Key key);
   Observable<Boolean> write(Key key, Raw raw);
}
Parser<Raw, Parsed> extends Func1<Raw, Parsed>{
    Parsed call(Raw raw);
}
```
---
## [fit]**Fetcher defines how a Store will get new data**
```java
Fetcher<Pizza,String> pizzaFetcher =
new Fetcher<Pizza, String>() {
   public Observable<Pizza> fetch(String topping) {
       return pizzaSource.fetch(topping);
};
```
---
# Becoming Observable

```java



Fetcher<Pizza,String> pizzaFetcher =
topping ->
Observable.fromCallable(() -> client.fetch(topping));
```
---
# Parsers help with Fetchers that don't return View Models

---
#[fit] Some Parsers Transform
```java
Parser<Pizza, PizzaBox> boxParser = new Parser<>() {
   public PizzaBox call(Pizza pizza) {
       return new PizzaBox(pizza);
   }
};
```
---
# [fit]Others read Streams
```java
Parser<BufferedSource, Pizza> parser = source -> {
   InputStreamReader reader =
   new InputStreamReader(source.inputStream());
   return gson.fromJson(reader, Pizza.class);
}
```
---

# [fit] MiddleWare is for the common JSON cases
```java
Parser<BufferedSource, Pizza> parser
= GsonParserFactory.createSourceParser(gson,Pizza.class)

Parser<Reader, Pizza> parser
= GsonParserFactory.createReaderParser(gson,Pizza.class)

Parser<String, Pizza> parser
= GsonParserFactory.createStringParser(gson,Pizza.class)

'com.nytimes.android:middleware:CurrentVersion'
'com.nytimes.android:middleware:jackson:CurrentVersion'
'com.nytimes.android:middleware-moshi:CurrentVersion'
```
---
#[fit]Data Flow with a Parser
![fit inline](/Users/Nakhimovich/Pictures/parser.png)

---
#[fit]Adding Offline with Persisters

---
#[fit] File System Record Persister
```java
FileSystemRecordPersister.create(
fileSystem,key -> "pizza"+key, 1, TimeUnit.DAYS);

```
---
# File System is KISS  storage
```java
 interface FileSystem {
   BufferedSource read(String var) throws FileNotFoundException;
   void write(String path, BufferedSource source) throws IOException;
   void delete(String path) throws IOException;
}

compile 'com.nytimes.android:filesystem:CurrentVersion'

```
---
## Don't like our persisters?
## **No Problem! You can implement your own**
---
#[fit] Persister interfaces
```java

Persister<Raw, Key> {
   Observable<Raw> read(Key key);
   Observable<Boolean> write(Key key, Raw raw);
 }

Clearable<Key> {
    void clear(Key key);
}

RecordProvider<Key> {
    RecordState getRecordState( Key key);
```
---
#[fit]Data Flow with a Persister
![fit inline](/Users/Nakhimovich/Pictures/persister.png)

---

![](store.png)
##We have our components!
##Let's build and open a store

---
# MultiParser
```java
List<Parser> parsers=new ArrayList<>();
parsers.add(boxParser);
parsers.add(GsonParserFactory.createSourceParser(gson, Pizza.class));
```
---
## Store Builder
```java
List<Parser> parsers=new ArrayList<>();
parsers.add(boxParser);
parsers.add(GsonParserFactory.createSourceParser(gson, Pizza.class));

StoreBuilder<String,BufferedSource,Pizza> parsedWithKey()

```
---
## Add Our Parser
```java
List<Parser> parsers=new ArrayList<>();
parsers.add(boxParser);
parsers.add(GsonParserFactory.createSourceParser(gson, Pizza.class));

StoreBuilder<String,BufferedSource,Pizza> parsedWithKey()
       .fetcher(topping -> pizzaSource.fetch(topping))

```
---
## Now Our Persister
```java
List<Parser> parsers=new ArrayList<>();
parsers.add(boxParser);
parsers.add(GsonParserFactory.createSourceParser(gson, Pizza.class));

StoreBuilder<String,BufferedSource,Pizza> parsedWithKey()
       .fetcher(topping -> pizzaSource.fetch(topping))
       .persister( FileSystemRecordPersister.create( fileSystem,
          key -> "prefix"+key, 1, TimeUnit.DAYS))

```
---
## And Our Parsers
```java
List<Parser> parsers=new ArrayList<>();
parsers.add(boxParser);
parsers.add(GsonParserFactory.createSourceParser(gson, Pizza.class));

StoreBuilder<String,BufferedSource,Pizza> parsedWithKey()
       .fetcher(topping -> pizzaSource.fetch(topping))
       .persister( FileSystemRecordPersister.create( fileSystem,
          key -> "prefix"+key, 1, TimeUnit.DAYS))
       .parsers(parsers)
```
---
## Time to Open a Store
```java
List<Parser> parsers=new ArrayList<>();
parsers.add(boxParser);
parsers.add(GsonParserFactory.createSourceParser(gson, Pizza.class));

Store<Pizza, String> pizzaStore =
StoreBuilder<String,BufferedSource,Pizza> parsedWithKey()
       .fetcher(topping -> pizzaSource.fetch(topping))
       .persister( FileSystemRecordPersister.create( fileSystem,
      key -> "prefix"+key, 1, TimeUnit.DAYS))
       .parsers(parsers)
       .open();
```
---
![](cash.jpg)
## [fit]Configuring caches
---
##[fit] Configuring Memory Policy
```java
StoreBuilder
       .<String, BufferedSource, Pizza>parsedWithKey()
       .fetcher(topping -> pizzaSource.fetch(topping))
       .memoryPolicy(MemoryPolicy
               .builder()
               .setMemorySize(10)
               .setExpireAfter(24)
               .setExpireAfterTimeUnit(HOURS)
               .build())
       .open()
```
---
##[fit] Refresh On Stale - BackFilling the Cache
```java
Store<Pizza, String> pizzaStore = StoreBuilder
       .<String, BufferedSource, Pizza>parsedWithKey()
       .fetcher(topping -> pizzaSource.fetch(topping))
       .parsers(parsers)
       .persister(persister)
       .refreshOnStale()
       .open();
```
---
##[fit] Network Before Stales Policy
```java
Store<Pizza, String> pizzaStore = StoreBuilder
       .<String, BufferedSource, Pizza>parsedWithKey()
       .fetcher(topping -> pizzaSource.fetch(topping))
       .parsers(parsers)
       .persister(persister)
       .networkBeforeStale()
       .open();
```
---
![](wild.jpg)
#[fit] Stores in the wild

---

#[fit] Intern Project: Best Sellers List
```java

public interface Api {
    @GET
    Observable<BufferedSource> 
    getBooks(@Path("category") String category);
```
---
```java
    @Provides 
    @Singleton 
    Store<Books, BarCode> provideBooks(FileSystem fileSystem, 
    Gson gson, Api api) {
        return StoreBuilder.<BarCode, BufferedSource, Books>
                 parsedWithKey()
                .fetcher(category -> api.getBooks(category))
                .persister(SourcePersisterFactory.create(fileSystem))
                .parser(GsonParserFactory.createSourceParser(gson, Books.class))
                .open();
    }

 ```
 ---
 ```java   
    public class BookActivity{
    ...
    onCreate(...){
      bookStore
        .get(category)
        .subscribe();
    }
```
---
# How do books get updated?
```java
    public class BackgroundUpdater{
      ...
     bookStore
        .fresh(category)
        .subscribe();
      }
```
---
^If the background update fails UI will fallback to network
#[fit] Data Available when screen needs it
#[fit] UI calls get, background services call fresh
---
![](dependency.jpg)
# Dependent Calls
---
#[fit] Simple case Using RxJava Operators
##Map  Store result to another Store Result
```java
public Observable<SectionFront> getSectionFront(@NonNull final String name) {
        return feedStore.get()
                .map(latestFeed -> latestFeed.getSectionOrBlog(name))
                .flatMap(section -> sfStore.get(SectionFrontId.of(section)));
    }
```
---
#[fit]More Complicated Example
### Video Store Returns single videos, Playlist Store returns playlist, 
###How do we combine?

---
![](636119392031889369953848753_customer-relationship.jpg)
#[fit] Relationships by overriding
#[fit] Get/Fetch
---
#[fit] Step 1: Create Video Store
```java
    Store<Video, Long> provideVideoStore(VideoFetcher fetcher,
                                         Gson gson) {
        return StoreBuilder.<Long, BufferedSource, Video>parsedWithKey()
                .fetcher(fetcher)
                .parser(GsonParserFactory
                           .createSourceParser(gson, Video.class))
                .open();
    }
```
---
#[fit] Step 2: Create Playlist Store
```java
    public class VideoPlaylistStore extends RealStore<Playlist, Long> {

    final Store<CherryVideoEntity, Long> videoStore;

    public VideoPlaylistStore(@NonNull PlaylistFetcher fetcher,
                              @NonNull Store<CherryVideoEntity, Long> videoStore,
                              @NonNull Gson gson) {
        super(fetcher, NoopPersister.create(),
                GsonParserFactory.createSourceParser(gson, Playlist.class));
        this.videoStore = videoStore;
    }
```
---
#[fit] Step 3: Override store.get()
```java
public class VideoPlaylistStore extends RealStore<Playlist, Long> {

    final Store<CherryVideoEntity, Long> videoStore;

    @Override
    public Observable<Playlist> get(Long playlistId) {
        return super.get(playlistId)
                .flatMap(playlist -> 
                  from(playlist.videos())
                        .concatMap(video -> videoStore.get(video.id()))
                        .toList()
                        .map(videos ->
                                builder().from(playlist)
                                        .videos(videos)
                                        .build()));
    }
```
---
![](listening.jpg)
#[fit] Listening for changes
---
###Step 1: Subscribe to Store, Filter what you need
```java
sectionFrontStore.subscribeUpdates()
                .observeOn(scheduler)
                .subscribeOn(Schedulers.io())
                .filter(this::sectionIsInView)
                .subscribe(this::handleSectionChange);
```

###Step 2: Kick off a fresh request to that store
```java
    public Observable<SectionFront> refreshAll(final String sectionName) {
        return feedStore.get()
                         .flatMapIterable(this::updatedSections)
                         .map(section->sectionFrontStore.fetch(section))
                         latestFeed -> fetch(latestFeed.changedSections));
    }
```
---
![](Assembly-Line.png)
## [fit]Bonus: How we made Store
---
## RxJava for API and Functional Niceties
```java
public Observable<Parsed> get(@Nonnull final Key key) {
        return Observable.concat(
            cache(key),
            fetch(key)
        ).take(1);
    }

    public Observable<Parsed> getRefreshing(@Nonnull final Key key) {
        return get(key)
            .compose(repeatOnClear(refreshSubject, key));
    }
```
---
## Guava Caches For Memory Cache
```
 memCache = CacheBuilder
            .newBuilder()
            .maximumSize(maxSize())
            .expireAfterWrite(expireAfter(),timeUnit())
            .build();
```
---
## In Flight Throttler Too
```java

  inflighter.get(key, new Callable<Observable<Parsed>>() {
                public Observable<Parsed> call() {
                    return response(key);
                }
            })


```
` compile 'com.nytimes.android:cache:CurrentVersion'`

###Why Guava Caches? 
###They will block across threads saving precious MBs of downloads when making same requests in parallel

---
## FileSystem:
###Built using OKIO to allow streaming from Network to disk & disk to parser
####On Fresh Install NY Times app successfully downloads and caches 60mb of data (insert emoji)
---
![](contributing.jpg)
# Would love contributions & feedback!
####thanks for listening
