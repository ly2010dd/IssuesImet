# JSON with GSON and abstract classes

---
Reference: http://ovaraksin.blogspot.com.es/2011/05/json-with-gson-and-abstract-classes.html

---

There is only one problem I have faced with abstract classes. Assume, we have an abstract class AbstractElement and many other classes extending this one

```
public abstract class AbstractElement {
    private String uuid;
 
    // getter / setter
}
 
public class Circle extends AbstractElement {
   ...
}
 
public class Rectangle extends AbstractElement {
   ...
}
 
public class Ellipse extends AbstractElement {
   ...
}
```
Assume now, we store all concrete classes in a list or a map parametrized with AbstractElement

```
public class Whiteboard
{
    private Map<String, AbstractElement> elements = 
            new LinkedHashMap<String, AbstractElement>();
    ...
}
```
The problem is that the concrete class is undisclosed during deserialization. It's unknown in the JSON representation of Whiteboard. How the right Java class should be instantiated from the JSON representation and put into the Map<String, AbstractElement> elements? I have nothing found in the documentation what would address this problem. It is obvious that we need to store a meta information in JSON representations about concrete classes. That's for sure. Gson allows you to register your own custom serializers and deserializers. That's a power feature of Gson. Sometimes default representation is not what you want. This is often the case e.g. when dealing with third-party library classes. There are enough examples of how to write custom serializers / deserializers. I'm going to create an adapter class implementing both interfaces JsonSerializer, JsonDeserializer and to register it for my abstract class AbstractElement.

```
GsonBuilder gsonBilder = new GsonBuilder();
gsonBilder.registerTypeAdapter(AbstractElement.class, new AbstractElementAdapter());
Gson gson = gsonBilder.create();
```
And here is AbstractElementAdapter:

```
package com.googlecode.whiteboard.json;
 
import com.google.gson.*;
import com.googlecode.whiteboard.model.base.AbstractElement;
import java.lang.reflect.Type;
 
public class AbstractElementAdapter implements JsonSerializer<AbstractElement>, JsonDeserializer<AbstractElement> {
    @Override
    public JsonElement serialize(AbstractElement src, Type typeOfSrc, JsonSerializationContext context) {
        JsonObject result = new JsonObject();
        result.add("type", new JsonPrimitive(src.getClass().getSimpleName()));
        result.add("properties", context.serialize(src, src.getClass()));
 
        return result;
    }
 
    @Override
    public AbstractElement deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context)
        throws JsonParseException {
        JsonObject jsonObject = json.getAsJsonObject();
        String type = jsonObject.get("type").getAsString();
        JsonElement element = jsonObject.get("properties");
 
        try {
            return context.deserialize(element, Class.forName("com.googlecode.whiteboard.model." + type));
        } catch (ClassNotFoundException cnfe) {
            throw new JsonParseException("Unknown element type: " + type, cnfe);
        }
    }
}
```
I add two JSON properties - one is "type" and the other is "properties". The first property holds a concrete implementation class (simple name) of the AbstractElement and the second one holds the serialized object itself. The JSON looks like

```
{
    "type": "Circle",
    "properties": {
        "radius": 10,
        "backgroundColor": "#FF0000",
        "borderColor": "#000000",
        "scaleFactor": 0.5,
        ...
    }
}
```
We benefit from the "type" property during deserialization. The concrete class can be instantiated now by Class.forName("com.googlecode.whiteboard.model." + type) where "com.googlecode.whiteboard.model." + type is a fully qualified class name. The following call

```
public <T> T deserialize(JsonElement json, Type typeOfT) throws JsonParseException
```
from JsonDeserializationContext invokes default deserialization on the specified object and completes the job.
