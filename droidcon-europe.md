# [fit] Store
# [fit] **Data Loading Made Easy**
# [fit] _**With Mike Nakhimovich, NY Times**_
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
# [fit] **We have top notch Fetchers, Parsers & Persisters**
---
#[fit] **Fetchers**
# [fit] ~~HTTPURLCONNECTION~~ Volley, Retrofit, Okhttp
---
#[fit] **Persisters**
#[fit] ~~Shared Prefs~~ Firebase, Realm, SqlBrite/SqlDelight
---
#[fit] **Parsers**
#[fit] ~~JSONObject~~ Jackson, Gson, Moshi
---
# [fit] Other Benefits
# [fit] of Open Source
—
# [fit] New Job,
# [fit] Same Libraries
---
# [fit] Fewer compromises
#[fit] doing consulting
---
# [fit] Shared knowledge
#[fit] in community
---
# [fit] Interns have resources
---

# [fit] What's NOT Easy?
#[fit] **DATA LOADING**
---
^Now raise your hand if you think the person sitting next to you does it in the same way
#[fit]  **What is Data Loading?**
###[fit] Data flow from network,
###[fit] through caches and transformations
---
##[fit] **Loading is complicated**
### How do I fetch?
### Where do I parse?
### When do I cache?
---
#[fit]**What makes it complicated?**
##  Rotation is a
## special snowflake
# **_:snowflake:_**
—

### Some Apps avoid it
### Others use deprecated APIs
### Loaders & Content Providers require an advanced degree to understand
### And the rest seem to serialize the world

---
# New York Times built **Store** to simplify data loading
#### github.com/NYTimes/store
---
# [fit]Goals
---
## Data should survive configuration changes
## agnostic of where it comes from or how it is needed
---
^Activities should deal with activity stuff, presenters should deal with presenting. Stores should be used to store data
## Activities and presenters should stop retaining data
### We should follow **single responsibility principles**


---
#[fit] Offline should not be an afterthought
### Caching should be standard,
## **not the exception**
---
^Our team becomes 50% bigger every summer
## API should be simple enough for an intern to use yet robust enough for every data load
---
# [fit] So what's our Data Story at NY Times?
---
#[fit] 80% case:
#[fit] Want data,
#[fit] don't care from where
---
#[fit] Other times,
## Want fresh data for background updates and pull to refresh
---
# [fit] Data is dependent on each other
# [fit] Data calls should be too
---
# [fit] Requests need to be async and reactive
# [fit] With a way to force a refresh
---
# Performance is important
### We load huge amounts of data for a lightweight app
---
## Parsing should be done efficiently and minimally
---
# Mission
### Create reactive, persistent, Data Stores to simplify data loading
---

## **Repository Pattern**
### Separate the logic that retrieves the data and maps it to the entity model from the [view] logic that acts on the model.
### The repository mediates between the data layer and the [view] layer of the application.
---

## **Why Repository?**
### Maximize the amount of code that can be tested with automation by isolating the data layer
---
## **Why Repository?**
###Makes it easier to have multiple devs working on same feature
---
## **Why Repository?**
### Data source from many locations will be centrally managed with consistent access rules and logic
---

## **Our Implementation**
### https://github.com/NYTimes/Store

—
# What is a **Store?**
###  A class that manages the fetching, parsing, and storage of a specific data model.
---
### [fit]**You Tell a Store:**
### [fit] How to fetch
### [fit] Where to cache
### [fit] How to parse
### [fit] **The Store manages the rest**

---
##How to use Stores
```java
Observable<T> get( V key);

Observable<T> fetch(V key)

Observable<T> getRefreshing(V key)

Observable<T> stream()

void clear(V key)

```
---
#80% Use Case
```java
public final class PizzaPresenter {
   Store<Pizza, String> store;
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
#[fit] Fetch when you want to skip cache
```java
public class PizzaPresenter {
   Store<Pizza, String> store;

   void onPTR() {
      store.fetch("veggie")
              .subscribe(pizza ->
              	getView().setData(pizza));
          }
      }
```
---
# [fit]How does Store route the request?
![fit inline](https://github.com/nytm/Store/raw/master/Images/store-2.jpg)

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
'com.nytimes.android:middleware:-jackson:CurrentVersion'
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
       .refreshOnStale()
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
