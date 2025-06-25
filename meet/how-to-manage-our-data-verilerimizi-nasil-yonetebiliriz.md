---
icon: head-side-goggles
---

# How To Manage Our Data? Verilerimizi Nasıl Yönetebiliriz?

## Description

What are common database solutions used in professional Unity projects? Which ones are best suited for my game?

Profesyonel Unity projelerinde yaygın olarak kullanılan veritabanı çözümleri nelerdir? Hangileri benim oyunum için en uygun olanlardır?

## Common Database Solutions

### 1. Unity's build-in

Use Unity’s built-in asset pipeline -- Scriptable Objects assets and sometimes .csv or .json files.

Unity’nin yerleşik varlık aktarım sistemini kullanın — Scriptable Object varlıkları ve bazen de .csv veya .json dosyaları.

### 2. Embedded Local Databases

Gömülü Yerel Veritabanları

1. **SQLite** (with sqlite-net, UniSQLite, etc.)
2. **LiteDB**

### 3. Cloud & Backend Services (Maybe all need spend <mark style="color:orange;background-color:orange;">money</mark>)

Bulut ve Arka Uç Hizmetleri (Belki hepsi ücretlidir)

1. **Unity Gaming Services (UGS)** (from unity)
2. **PlayFab** (from unity)
3. **Firebase (Firestore / Realtime DB)** (from google)
4. **Custom REST API + SQL/NoSQL** (A lot of App we can use, for example MySQL)

## In Production Phase (Runtime)

(<mark style="color:purple;">After development is complete and the game is running on players’ phones…)</mark>

### 1. Unity Build-in

Use for: Character profile pools, question pools data.

### 2. Embedded Local Database

**Use for**: Player progress, local caching of leaderboards, offline quiz history, <mark style="color:purple;">player himself's statistic data</mark>.&#x20;

Kullanım amacı: Oyuncu ilerlemesi, lider tablosunun yerel önbelleğe alınması, çevrimdışı quiz geçmişi, oyuncuya ait istatistik verileri.

> **GPT Recommendation**: If you expect significant offline play or want advanced querying, start with **SQLite** via the [sqlite-net](https://github.com/praeclarum/sqlite-net) wrapper.  Otherwise, if your only local needs are “last session stats” or caching a few values, Unity’s **PlayerPrefs** or a small JSON file will suffice.&#x20;
>
> **GPT Önerisi**: Eğer ciddi miktarda çevrimdışı oynanış bekliyorsanız veya gelişmiş sorgulama yapmak istiyorsanız, [sqlite-net](https://github.com/praeclarum/sqlite-net) sarmalayıcısı aracılığıyla **SQLite** ile başlayın. Aksi takdirde, yerel ihtiyaçlarınız yalnızca “son oturum istatistikleri” veya birkaç değeri önbelleğe almakla sınırlıysa, Unity’nin **PlayerPrefs** sistemi ya da küçük bir JSON dosyası yeterli olacaktır.

### 3. **Cloud & Backend Services** (Maybe all need spend money)

Mandatory once we have a <mark style="color:green;">**global leaderboard**</mark> <mark style="color:green;"></mark><mark style="color:green;">data</mark>. Küresel bir lider tablosu verisine sahip olduğumuzda zorunludur.

## In Development Phase

Some static data, such as information for 900 characters, may never change at runtime but often changes during development. Can we store it in a local embedded database or cloud service to make it easier for developers to update?

Bazı statik veriler, örneğin 900 karaktere ait bilgiler, çalışma zamanında asla değişmese de geliştirme sürecinde sık sık güncellenir. Geliştiricilerin bu verileri daha kolay güncelleyebilmesi için bunları yerel gömülü bir veritabanında veya bulut servisinde saklayabilir miyiz?

We can use Unity’s ScriptableObjects or CSV files starting in the production stage.

### Character Pool

The Character Pool in our game functions like NPC templates in other games -- belong to static data.

Oyunumuzdaki Karakter Havuzu, diğer oyunlardaki NPC şablonları gibi işlev görür — bu veriler statik veri kategorisine girer.

#### 1. Unity Build-in Scriptable Objects

Pro: No need to learn new things.&#x20;

Cons: <mark style="color:red;">Each time</mark> you update a character’s profile or a distractor (confusing) answer in the CSV, you must clear the ScriptableObject pool and reimport all characters.

#### 2. Embedded Local Database

Pro: Don't need clear pool and reimport&#x20;

Cons: I’m not sure if this can be version-controlled—<mark style="color:red;">how can we keep it in sync between Berke’s and Yuan’s computers?</mark>

#### 3. Cloud/Backend Services

Pro: Overcome the limitations of the two methods above…。。。&#x20;

Cons: Money
