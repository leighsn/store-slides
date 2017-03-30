# [fit] Store
# [fit] _**Data Loading Made Easy**_
# [fit] **Mike Nakhimovich - NY Times**
---

^NY Times App, Consulting Apps, Hack Day Apps, Apps for Blog Posts and Talks

# [fit] We _**:heart:**_
# [fit] Making Apps
---
#[fit] We _**:heart:**_
#[fit] Open Source
---
# [fit] Why do we **:heart:** it?
# [fit] Open Source Makes Life Easier
---
#Developers Are Lazy
# [fit] Open Source has simplified our lives
---
#[fit] Networking is easier through better Fetchers
# [fit] ~~**HTTPURLCONNECTION**~~ 
# [fit] Volley, Retrofit, Okhttp
---
#[fit] Data Storage/Persisters for Offline
#[fit] ~~**Shared Preferences**~~ 
#[fit] Firebase, Realm, SqlBrite/SqlDelight
---
#[fit] Parsers for Transformations
#[fit] ~~**JSONObject**~~ 
#[fit] Jackson, Gson, Moshi
---
# [fit] How does this 
# [fit] **benefit** you?
—
# [fit] You don't need to reinvent the wheel
# [fit] New Job,
# [fit] **Same** Libraries
---
#[fit] Shared knowledge in **community**
# [fit] We have 
# [fit] resources to look at
---

#[fit] What's NOT Easy?
#[fit] **DATA LOADING**
#[fit] Everyone does it differently
---
^Now raise your hand if you think the person sitting next to you does it in the same way
#[fit]  **What is Data Loading?**
# The Act of getting data from an external system to a user's screen
---
^We know that we're gonna use these great network libraries but what is actually doing the network call? does my activity really need to make it?
#[fit] **Loading is complicated** because
#[fit] Rotation Handling is a special 
#[fit]snowflake  **_:snowflake:_**
---
^there are apps with 100m downloads that don't handle rotation
### Some Apps avoid it
---
^getLastCustomNonConfigurationInstance
### Some Apps avoid it
### Others use deprecated APIs
—

### Some Apps avoid it
### Others use deprecated APIs
### Loaders & Content Providers require an advanced degree to understand
—

### Some Apps avoid it
### Others use deprecated APIs
### Loaders & Content Providers require an advanced degree to understand
### Serializing Presenters is not scalable

---
# New York Times built **Store** to simplify data loading
## github.com/NYTimes/store
---
# [fit] **Goals**
---
# **Data** should survive configuration changes,
#[fit] agnostic of where it comes from or how it is needed
---
^Activities should deal with activity stuff, presenters should deal with presenting. Stores should be used to store data
# Activities and presenters should stop retaining MBs of data.
#[fit] follow **single responsibility principles**


---
# Offline as a configuration
#[fit]Caching should be the standard not the exception!
#[fit] Developer should be able to make something cachable with 
---
^Our team becomes 50% bigger every summer
#API should be **simple** enough for an intern to use, yet **robust** enough for every data load.
---
# [fit] Our Use Cases
# [fit]Data Story at **NY Times**
---
#[fit] **80% case**:
# Need data, don't care if it fresh or cached
---
#[fit] Fetch from outside
## Want fresh data: 
##**background updates** and **pull to refresh**
---
# Data is dependent on each other,
# Data calls should be too
##For example: Anytime we fetch an article we want comments too
---
# [fit] Requests need to be 
#[fit]**async** & **reactive**
---
# Performance is important
## We load **HUGE** amounts of data for a small app
###New calls should hook into in flight responses
---
# **Parsing** should be done efficiently and minimally
#[fit] Parse once and cache the result for future calls
---
# How did we achieve our Goals?
###By creating reactive and persistent Data Stores that follow the Repository Pattern
---

#[fit] **Repository Pattern?**
### Separate the logic that retrieves the data and maps it to the [view] model from the [view] logic that acts on the model.
### The repository mediates between the data layer and the [view] layer of the application.
---

# **Why Repository?**
# Maximize the amount of code that can be tested with automation by isolating the data layer
---
# **Why Repository?**
#Makes it easier to have multiple devs working on same feature
---
# **Why Repository?**
# Data source from many locations will be centrally managed with consistent access rules and logic
---

# **Our Implementation**
## https://github.com/NYTimes/Store
###(How'd we do?)
—
# What is a **Store?**
## A class that manages the fetching, parsing, and storage of a specific data model
#### Stores bring together all those great open source libs from earlier
---
# [fit]**You Tell a Store:**
#  What to **fetch**
---
# [fit]**You Tell a Store:**
#  What to **fetch**
#  Where to **cache**
---
# [fit]**You Tell a Store:**
#  What to **fetch**
#  Where to **cache**
# How to **parse**
---
# [fit]**You Tell a Store:**
#  What to **fetch**
#  Where to **cache**
# How to **parse**
# [fit] The Store manages the rest

---
##Stores Have An Observable API
```java
Observable<T> get( V key);

Observable<T> fetch(V key)

Observable<T> getRefreshing(V key)

Observable<T> stream()

void clear(V key)

```
---
#Handling our 80% Use Case
```java
public final class PizzaPresenter {
   Store<Pizza, String> pizzaStore;
   void onLoad() {
       store.get("cheese")
               .subscribe(pizza -> getView()
               .setData(pizza));
       }
   }
```
---  
# [fit]How does Store route the request?
 ![inline fit](https://github.com/nytm/Store/raw/master/Images/store-1.jpg)  

---
#Handling Configuration Change
## Save/Restore only your 🧀 and get your 🍕  
```java
public final class PizzaPresenter {
   Store<Pizza, String> pizzaStore;
   void onAfterRotate(String topping) {
       store.get(topping)
               .subscribe(pizza -> getView()
               .setData(pizza));
       }
   }
```
##Stores retain data UI Maintains Keys
---
# What we gain?:
###Presenters don't need to be retained
###Don't need to worry about TransactionTooLargeExceptions
###Data Doesn't need to be Parcelable

---

##Block if not present:
```java
for(i=0;i<20;i++){
   store.get(topping)
               .subscribe(pizza -> getView()
               .setData(pizza));
}
```
##Many concurrent Get requests will still only hit network once

---


#[fit] Fetch when you want to skip cache
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
![fit inline](https://github.com/nytm/Store/raw/master/Images/store-2.jpg)
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
# [fit]How does Store route the request?
# [fit]Insert image for Stream
---
# [fit] Get Refreshing - Update on Clear

```java
public class PriceList {
   Store<List<Double>, String> prices;
   void pizzaPrices() {
       store
          .getRefreshing("prices")
          .subscribe(prices -> display(prices));
      }

  void updatePrices(){
	api.updatePizzaPrices(prices)
	   .doOnNext(response->store.clear("pizza");
	}

```
---

## [fit] How does Store route the request?
## [fit] Insert image for Stream
---
## [fit] **To Build a Store**
### Implement and supply Interfaces to a Builder which opens a Store with a consistent Public API.
---
# **Interfaces**
```java
Fetcher<Raw, Key>
   Observable<Raw> fetch(Key key);

Persister<Raw, Key>
   Observable<Raw> read(Key key);
   Observable<Boolean> write(Key key, Raw raw);

Parser<Raw, Parsed> extends Func1<Raw, Parsed>
 	  Parsed call(Raw raw);
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
# Wrap synchronous client in Observable
```java
Fetcher<Pizza,String> pizzaFetcher =
topping ->
Observable.fromCallable(() -> client.fetch(topping));
```
---
## [fit] Since APIs don't return POJOs
## [fit] we will implement
## [fit] Data Parsers
---
# [fit] Some Parsers read streams
```java
Parser<BufferedSource, Pizza> parser = source -> {
   InputStreamReader reader =
   new InputStreamReader(source.inputStream());
   return gson.fromJson(reader, Pizza.class);
}
```
---
#[fit] Others make View Models
```java
Parser<Pizza, PizzaBox> boxParser = new Parser<>() {
   public PizzaBox call(Pizza pizza) {
       return new PizzaBox(pizza);
   }
};
```
---
# [fit] MiddleWare is for the common cases
```java
Parser<BufferedSource, Pizza> parser
= GsonParserFactory.createSourceParser(gson,Pizza.class)



'com.nytimes.android:middleware:CurrentVersion'
'com.nytimes.android:middleware:jackson:CurrentVersion'
'com.nytimes.android:middleware-moshi:CurrentVersion'

```
---
#[fit]Data Flow with a Parser
![fit inline](https://github.com/nytm/Store/raw/master/Images/store-3.jpg)

---
#[fit]Remember we are offline first!
#[fit] Let's get a Persister
---
#[fit] File System Record Persister
```java
FileSystemRecordPersister.create(fileSystem,
key -> "prefix"+key
1, TimeUnit.DAYS);

public interface PathResolver<T> {
   String resolve( T key);
}
```
---
# File System!
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
![fit inline](https://github.com/nytm/Store/raw/master/Images/store-5.jpg)

---
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
## [fit]Configure for use case
---
##[fit] Configuring Memory Policy
```java
StoreBuilder
       .<String, BufferedSource, Pizza>parsedWithKey()
       .fetcher(topping -> pizzaSource.fetch(topping))
       .memoryPolicy(MemoryPolicy
               .builder()
               .setMemorySize(10)
               .setExpireAfter(TimeUnit.HOURS.toSeconds(24))
               .setExpireAfterTimeUnit(TimeUnit.SECONDS)
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
## [fit]Bonus: How we made Store
---
## RxJava for API and Functional Concepts
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
## Guava Caches for in Memory Cache
```
 memCache = CacheBuilder
            .newBuilder()
            .maximumSize(maxSize())
            .expireAfterWrite(expireAfter(),timeUnit())
            .build();
```
---
## And For Inflight Throttler
```java
inFlightRequests = CacheBuilder
                .newBuilder()
                .build();

```
` compile 'com.nytimes.android:cache:CurrentVersion'`

###Why Guava Caches? They will block across threads saving precious MBs of downloads when making same requests in parallel

---
## FileSystem:
###Built using OKIO to allow streaming from Network to disk & disk to parser
####On Fresh Install NY Times app successfully downloads and caches 60mb of data (insert emoji)

---
##Finishing Thoughts
### *Make Stores Singleton and share them
### *UI should maintain only keys to data
### *Stores can be subclasses (RealStore)
### *Mention stores depending on each other?
### *Would love contributions!
