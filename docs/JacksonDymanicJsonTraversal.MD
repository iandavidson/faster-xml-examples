# Using FasterXML-Jackson on dynamically structured JSON 

## Goals:
- Traverse all elements within a given json document, with no prior knowledge of structure
- Find the sum of every number in the document and return.  

###  Set up: 
- Java version 21
- jackson-databind

```yaml
# build.gradle
dependencies {
    ...
    implementation 'com.fasterxml.jackson.core:jackson-databind:2.17.2'
}
```

### Step by step:

#### Construct ObjectMapper instance:
ObjectMapper will be used to interpret the JSON document.

For this example, since we have no expectations of conventional structure, etc. 
We simply use ObjectMapper with default configuration. Using the following line:

```java
        ObjectMapper objectMapper = new ObjectMapper();
```

#### Construct JsonNode instance from JSON document via ObjectMapper:
```java
        JsonNode jsonNode = null;
        try {
            jsonNode = objectMapper.readTree(json);
            return findSum(jsonNode);
        } catch (JsonMappingException e) {
            throw new RuntimeException(e);
        } catch (JsonProcessingException e) {
            throw new RuntimeException(e);
        } catch (IllegalStateException e) {
            throw new RuntimeException(e);
        }
```

#### Traverse JSON document using JsonNode instance, sum up numbers encountered:
A given JsonNode can represent many different mediums, such as {Array, Object, Number, String, etc.}. 

We leverage the Enum type, JsonNodeType, provided by Jackson to differentiate what type of node we are currently at.

Here's how the difference cases breakdown:
 - Array:  call the method `elements()`, to find all inner JsonNode objects, then recursively call on each
 - Object: call the method `properties()`, to find all key-value pairs within jsonNode, recursively call on each value
 - Number: interpret integer value of JsonNode with the method `asInt()`
 - String: in this scenario we don't plan to attempt to interpret string fields as numerical values, we simply fall through this case

```java
private long findSum(final JsonNode jsonNode) {
        long sum = 0L;
        JsonNodeType type = jsonNode.getNodeType();
        
        switch (type) {
            case ARRAY -> {
                for (Iterator<JsonNode> it = jsonNode.elements(); it.hasNext(); ) {
                    JsonNode child = it.next();
                    sum += computeSumPart1(child, part2);
                }
            }
            case OBJECT -> {
                for (Map.Entry<String, JsonNode> child : jsonNode.properties()) {
                    sum += computeSumPart1(child.getValue(), part2);
                }
            }
            case NUMBER -> {
                sum += jsonNode.asInt();
            }
            case STRING -> {
                //do nothing
            }
            case null, default -> {
                throw new IllegalStateException();
            }
        }

        return sum;
    }
```







