# api-swgoh-help [![Maven Central](https://img.shields.io/maven-central/v/help.swgoh.api/swgoh-api-connector.svg?style=flat-square)](https://mvnrepository.com/artifact/help.swgoh.api/swgoh-api-connector)
Java client wrapper for the API at https://api.swgoh.help

Due to the extreme versatility of this API and the available options that are given that modify the response structures, all of the responses returned by each endpoint will be raw JSON in the form of a `java.lang.String` for now. The consumer of this API client is then free to parse the data into whatever form they choose.

# Usage
Include the swgoh-api-connector artifact:

`@VERSION@` should be replaced with the version you want to use.

### Maven
```xml
<dependency>
    <groupId>help.swgoh.api</groupId>
    <artifactId>swgoh-api-connector</artifactId>
    <version>@VERSION@</version>
</dependency>
```

### Gradle
```groovy
compile 'help.swgoh.api:swgoh-api-connector:@VERSION@'
```

### Examples
In code, use the SwgohAPIBuilder to initialize a new instance of the client:
```java
SwgohAPI api = new SwgohAPIBuilder()
        .withUsername( "username" )
        .withPassword( "password" )
        .build();
```

Request for a player profile by ally code:
```java
//single player
int allyCode = 123456789;
String player = api.getPlayer( allyCode ).get();
```
```java
//multiple players
int allyCode = 123456789;
int otherAllyCode = 987654321;
int[] allyCodes = new int[] { allyCode, otherAllyCode };
String players = api.getPlayers( allyCodes ).get();
```
```java
//only return certain fields in the response
int allyCode = 123456789;
String player = api.getPlayer( 
        allyCode,
        SwgohAPI.PlayerField.name,
        SwgohAPI.PlayerField.allyCode,
        SwgohAPI.PlayerField.roster,
).get();
```

Request guild info by ally code:
```java
int allyCode = 123456789;
String guild = api.getGuild( allyCode ).get();
```
```java
//only return certain fields in the response
int allyCode = 123456789;
String guild = api.getGuild( 
        allyCode,
        SwgohAPI.GuildField.name,
        SwgohAPI.GuildField.gp,
        SwgohAPI.GuildField.roster,
        SwgohAPI.GuildField.updated
).get();
```

Request various other kinds of data:
```java
//no filtering, large response
String unitsJson = api.getSupportData( SwgohAPI.Collection.unitsList ).get();
```
```java
//specify custom filtering criteria for a smaller response
Map<String, Object> matchCriteria = new HashMap<>();
matchCriteria.put( "baseId", "GREEDO" );
matchCriteria.put( "rarity", 7 );
String greedoJson = api.getSupportData( SwgohAPI.Collection.unitsList,
        matchCriteria,
        SwgohAPI.Language.English,
        "nameKey", "combatType", "descKey", "thumbnailName", "baseId"
).get();
```
...and so much more! Please reference `SwgohAPI.Collection` for a list of all available data collections.

# Global Discord Registry
##### (Patreon-tier API accounts only)
This API offers a Discord registry, which is simply a map of ally codes to Discord IDs.

This registry is shared among all Patreon-tier API accounts. If your users have already registered through a different tool that uses this registry, then their registration will already be available to you as well. 

Registering an ally code more than once will cause that ally code's association to be **overwritten**. Registering a Discord ID more than once will cause that Discord ID to be **linked to multiple ally codes**. This allows users to link their Discord account to all of their game alts, but does not allow a game account to be tied to more than one Discord user at a time.

```java
//create association
int allyCode = 123456789;
String discordId = "098765432123456789";
api.register( allyCode, discordId );
```
```java
//get association
int allyCode = 123456789;
RegistrationResponse response = api.getRegistrationByAllyCode( allyCode ).get();
//or
String discordId = "098765432123456789";
RegistrationResponse response = api.getRegistrationByDiscordId( discordId ).get();
```
```java
//remove association
int allyCode = 123456789;
api.unregisterAllyCode( allyCode );
//or
String discordId = "098765432123456789";
api.unregisterDiscordId( discordId );
```

In order to use this feature, your API account will need to upgrade to Patreon-tier.

You can do this by becoming a Patreon patron and supporting the API in any of the following ways:<br/>
[API Development](https://www.patreon.com/shittybots/overview)<br/>
[Server Hosting](https://www.patreon.com/user/posts?u=470177)

After becoming a patron, be sure to log into Discord and contact either `Midgar777#0394` or `shittybill#3024` and give them your API account username so that your account can be given access to this awesome feature.

# Asynchronous API with CompletableFuture
Don't want to wait on a response from the API? Set up a callback method using the returned [CompletableFuture](https://www.baeldung.com/java-completablefuture).

```java
int allyCode = 123456789;
api.getPlayer( allyCode ).thenAccept( player -> {
    //do something with player. This will happen in the future, leaving the main thread unblocked
});
```

# Language Specification
To receive data in a supported language, simply pass the specified language into the overloaded method of your choice.

Example of language-specified result:
```java
String jsonUnitsInKorean = api.getSupportData( SwgohAPI.Collection.unitsList, SwgohAPI.Language.Korean );
```

# Image API
Customized images of any unit in the game can be obtained through the construction of either a `ToonImageRequestBuilder` or a `ShipImageRequestBuilder`.

```java
//raw bytes of an image
String baseId = "DARTHTRAYA"; //a full list of baseIds can be obtained through the "unitsList" collection
byte[] bytes = api.getImage( 
        new ToonImageRequestBuilder( baseId )
        .withStars( 7 )
        .withLevel( 85 )
        .withGear( 12 )
        .build()
).get();
```
```java
//image in the form of a BufferedImage
String baseId = "TIEREAPER"; //a full list of baseIds can be obtained through the "unitsList" collection
BufferedImage image = api.getBufferedImage(
        new ShipImageRequestBuilder( baseId )
        .withLevel( 50 )
        .withStars( 5 )
        .addPilot( "DEATHTROOPER", 85, 11, 7, null )
        .addPilot( "SHORETROOPER", 75, 5, 5, null )
        .build()
).get();
```

# Spring Integration
If you'd like to see this work with [Spring's dependency injection magic](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#spring-core), simply write something similar to the following code:
```java
@Configuration
public class SwgohConfiguration
{
    @Bean
    public SwgohAPI swgohApi()
    {
        return new SwgohAPIBuilder()
                .withUsername( "username" )
                .withPassword( "password" )
                .build();
    }
}
```

And it's as simple as that. Now you can `@Autowire` the API client to your heart's content.
```java
@Service
public class SwgohService
{
    @Autowired
    private SwgohAPI swgohApi;
}
```

# Available Language Clients
NodeJS:<br/>
https://github.com/r3volved/api-swgoh-help/tree/node

Java:<br/>
https://github.com/j0rdanit0/api-swgoh-help

C#:<br/>
https://github.com/SdtBarbarossa/SWGOH-Help-Api-C-Sharp
https://gist.github.com/dwcullop/9c6b7933fe23163e59b94da1997adee7

PHP:<br/>
https://github.com/r3volved/api-swgoh-help/tree/php <br/>
https://github.com/rayderua/swhelp-api-client

Google App Script:<br/>
https://github.com/Dragonsight91/api-swgoh-help

Python:<br/>
https://github.com/platzman/swgoh.help.python
