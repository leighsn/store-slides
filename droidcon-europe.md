# [fit] Store
# [fit] **Data Loading Made Easy**
# [fit] _**With Mike Nakhimovich, NY Times**_
---
# [fit] Common Problems in Android
## [fit] How to Solve them with Store
---
^NY Times App, Consulting Apps, Hack Day Apps, Apps for Blog Posts and Talks

# [fit] We _**:heart:**_
# [fit] Making Apps
---

# [fit] Open Source Makes
# [fit] Life Easier

---
#[fit] Networking
# [fit] Volley, Retrofit, Okhttp
---
#[fit] Storage
#[fit] Firebase, Realm, SqlBrite
---
#[fit] Parsers
#[fit] Jackson, Gson, Moshi

---
#[fit] :heart_eyes:
#[fit] Why We Love
#[fit]Open Source
---
# [fit] Standardization
# [fit] Makes onboarding easier
---
# [fit] New Job,
#[fit] Same Patterns
---
# [fit] Fewer compromises doing consulting
---
# [fit] Blogs Can Assume Knowledge
---
# [fit] Interns Have Resources
---
# [fit] What's NOT Standard?
#[fit] DATA LOADING
---
^Now raise your hand if you think the person sitting next to you does it in the same way
#[fit] How does your UI
#[fit] load and retain data?
---
#[fit] Activity as
#[fit] Data Manager
---
#[fit]The Good:
#[fit]Simple Rotation:
#[fit]
```java
getLastNonConfigurationInstance()
```
---
#[fit]The Bad:
#[fit]Bundle size limits
```java
TransactionTooLargeException
```
:scream:

---

# Presenter as Requester and Retainer

---
#[fit]Handle Rotation
# Serialize the Presenter?
# Make global and rebind?

---

> Rotation is a special snowflake **_:snowflake:_**
--Me

---
# New York Times built Store to simplify our data operations
# github.com/NYTimes/store
---
# [fit]Goals
---
# Data should survive configuration changes

---
# Agnostic of where it comes from or how it is needed
---
^Activities should deal with activity stuff, presenters should deal with presenting. Stores should be used to store data
# Activites and presenters should stop retaining data

---

# We will follow single responsibility principles
---
#[fit] standardize caching
# We want a simple way to fetch from cache. If not there, we get from network
---
#[fit] Offline First
# Need a way to pre-fetch and cache data
---
^Our team becomes 50% bigger every summer
#[fit] KISS
# API should be simple enough for an intern to use
---
# Need to be robust
# Stores should be flexible enough that devs can use in any app they build
---
# [fit] How does our UI
# [fit]interact with data?
---
#[fit] 80%
#[fit] Want data,
#[fit] don't care from where
---
#[fit] Other Times
# Want fresh data
---
