## Data for CA-2

For setting up zip code data you can load it from the following api: https://api.dataforsyningen.dk/postnumre  
Then to read this into your java program you can use the following code example:

```java
String jsonStr = "";
    try {
        URL url = new URL(urlString);
        HttpURLConnection con = (HttpURLConnection) url.openConnection();
        con.setRequestMethod("GET");
        con.setRequestProperty("Accept", "application/json");
        con.setRequestProperty("User-Agent", "MyApp");
        try (Scanner scan = new Scanner(con.getInputStream())) {
            while (scan.hasNext()) {
                jsonStr += scan.nextLine();
            }
        }
    } catch (IOException ioex) {
        System.out.println("Error: " + ioex.getMessage());
    }
    return jsonStr;
```
You can then load the json data into a DTO designed for the purpose.

In this repo you will also find two scripts you can use to set up zip code data for CA-2. This can be found in the file: _zipScript.sql_ you must use (unless you find a similar/better script). Additionally you can load data about hobbies from _hobbyScript.sql_.

### Script for zip-codes

For an entity class defined as below:

```java
@Entity
public class CityInfo implements Serializable {
    private static final long serialVersionUID = 1L;
    @Id
    @Column(length = 4)
    private String zipCode;
    @Column(length=35)
    private String city;
    ...
```

You can set up all Danish zip-codes using the script-file _zipScript.sql_

Either run the script your usual way, up against your database, or let JPA execute the script by adding these two lines to (only) the persistence-unit "pu" in persistence.xml.

````
<property name="javax.persistence.sql-load-script-source" value="META-INF/zipScript.sql"/>
<property name="javax.persistence.schema-generation.database.action" value="drop-and-create"/>
````

Note: this requires you to place the script under META-INF, (next to persistence.xml)

__After having executed this script, you should never write to the `CityInfo-table`, only READ.__
_(This is NOT something you have to enforce on the database, just realize that, once initialized, all data is here,  and they wont change)_

### Script for Hobbies

A problem you probably will find with the Hobby table is how to ensure that we won't get the same hobby in many different versions (Fodbold, fodbold, fådbold,football, soccer etc.).
One way to ensure that, is to only allow Hobbies selected from a list of approved hobbies.

The (non-normalized) script _hobbyScript.sql_ is designed from a screen-scraped version of the data found on this [Wikipeda-page](https://en.wikipedia.org/wiki/List_of_hobbies) and titles (name) are hereafter auto translated to danish (so be prepared for some "strange" hobbies)

You can use it if you set up your Hobby entity class to match the following:

```java
@Entity
public class Hobby implements Serializable {
    private static final long serialVersionUID = 1L;
  
    @Id
    @Column(length = 50)
    private String name;
  
    private String wikiLink;
    private String category;
    private String type;
    ...
```

Run the script, using one of the strategies given for zipScript.sql
