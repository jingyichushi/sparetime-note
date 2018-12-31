# JsonPath学习



> 引用

最近项目中需要进行单元测试，主要是对Controller类的方法进行单元测试，测试的结果是返回json体，因此需要对返回的json进行解析（而非反序列化）。这时发现了JsonPath，能够满足单元测试对json进行检查的需求。



**1.maven依赖**

```xml
<dependency>
    <groupId>com.jayway.jsonpath</groupId>
    <artifactId>json-path</artifactId>
    <version>2.4.0</version>
</dependency>
```



**2.基本知识**

| Operator                  | Description                                                  |
| ------------------------- | ------------------------------------------------------------ |
| `$`                       | The root element to query. This starts all path expressions. |
| `@`                       | The current node being processed by a filter predicate.      |
| `*`                       | Wildcard. Available anywhere a name or numeric are required. |
| `..`                      | Deep scan. Available anywhere a name is required.            |
| `.<name>`                 | Dot-notated child                                            |
| `['<name>' (, '<name>')]` | Bracket-notated child or children                            |
| `[<number> (, <number>)]` | Array index or indexes                                       |
| `[start:end]`             | Array slice operator                                         |
| `[?(<expression>)]`       | Filter expression. Expression must evaluate to a boolean value. |

| Function | Description                                                  | Output  |
| -------- | ------------------------------------------------------------ | ------- |
| min()    | Provides the min value of an array of numbers                | Double  |
| max()    | Provides the max value of an array of numbers                | Double  |
| avg()    | Provides the average value of an array of numbers            | Double  |
| stddev() | Provides the standard deviation value of an array of numbers | Double  |
| length() | Provides the length of an array                              | Integer |

| Operator | Description                                                  |
| -------- | ------------------------------------------------------------ |
| ==       | left is equal to right (note that 1 is not equal to '1')     |
| !=       | left is not equal to right                                   |
| <        | left is less than right                                      |
| <=       | left is less or equal to right                               |
| >        | left is greater than right                                   |
| >=       | left is greater than or equal to right                       |
| =~       | left matches regular expression [?(@.name =~ /foo.*?/i)]     |
| in       | left exists in right [?(@.size in ['S', 'M'])]               |
| nin      | left does not exists in right                                |
| subsetof | left is a subset of right [?(@.sizes subsetof ['S', 'M', 'L'])] |
| anyof    | left has an intersection with right [?(@.sizes anyof ['M', 'L'])] |
| noneof   | left has no intersection with right [?(@.sizes noneof ['M', 'L'])] |
| size     | size of left (array or string) should match right            |
| empty    | left (array or string) should be empty                       |



**3.测试json串**

```json
{
    "store": {
        "book": [
            {
                "category": "reference",
                "author": "Nigel Rees",
                "title": "Sayings of the Century",
                "price": 8.95
            },
            {
                "category": "fiction",
                "author": "Evelyn Waugh",
                "title": "Sword of Honour",
                "price": 12.99
            },
            {
                "category": "fiction",
                "author": "Herman Melville",
                "title": "Moby Dick",
                "isbn": "0-553-21311-3",
                "price": 8.99
            },
            {
                "category": "fiction",
                "author": "J. R. R. Tolkien",
                "title": "The Lord of the Rings",
                "isbn": "0-395-19395-8",
                "price": 22.99
            }
        ],
        "bicycle": {
            "color": "red",
            "price": 19.95
        }
    },
    "expensive": 10
}
```



controller方法：

```java
    @GetMapping("json")
    public String jsonPathTest(@PathVariable("path") String path){
        String json = "{\\\"store\\\":{\\\"book\\\":[{\\\"category\\\":\\\"reference\\\",\\\"author\\\":\\\"Nigel Rees\\\",\\\"title\\\":\\\"Sayings of the Century\\\",\\\"price\\\":8.95},{\\\"category\\\":\\\"fiction\\\",\\\"author\\\":\\\"Evelyn Waugh\\\",\\\"title\\\":\\\"Sword of Honour\\\",\\\"price\\\":12.99},{\\\"category\\\":\\\"fiction\\\",\\\"author\\\":\\\"Herman Melville\\\",\\\"title\\\":\\\"Moby Dick\\\",\\\"isbn\\\":\\\"0-553-21311-3\\\",\\\"price\\\":8.99},{\\\"category\\\":\\\"fiction\\\",\\\"author\\\":\\\"J. R. R. Tolkien\\\",\\\"title\\\":\\\"The Lord of the Rings\\\",\\\"isbn\\\":\\\"0-395-19395-8\\\",\\\"price\\\":22.99}],\\\"bicycle\\\":{\\\"color\\\":\\\"red\\\",\\\"price\\\":19.95}},\\\"expensive\\\":10}";
        return  JsonPath.read(json, path);
    }
```



测试结果：

所有书的作者：$.store.book[*].author

```json
[
   "Nigel Rees",
   "Evelyn Waugh",
   "Herman Melville",
   "J. R. R. Tolkien"
]
```



所有作者（不仅仅是书的）：$..author

```json
[
   "Nigel Rees",
   "Evelyn Waugh",
   "Herman Melville",
   "J. R. R. Tolkien"
]
```



store下的所有东西：$.store.*

```json
[
   [
      {
         "category" : "reference",
         "author" : "Nigel Rees",
         "title" : "Sayings of the Century",
         "price" : 8.95
      },
      {
         "category" : "fiction",
         "author" : "Evelyn Waugh",
         "title" : "Sword of Honour",
         "price" : 12.99
      },
      {
         "category" : "fiction",
         "author" : "Herman Melville",
         "title" : "Moby Dick",
         "isbn" : "0-553-21311-3",
         "price" : 8.99
      },
      {
         "category" : "fiction",
         "author" : "J. R. R. Tolkien",
         "title" : "The Lord of the Rings",
         "isbn" : "0-395-19395-8",
         "price" : 22.99
      }
   ],
   {
      "color" : "red",
      "price" : 19.95
   }
]
```



store节点下的所有价格：$.store..price

```json
[
   8.95,
   12.99,
   8.99,
   22.99,
   19.95
]
```



第三本书：$..book[2]或者$..book[2:3]

```json
[
   {
      "category" : "fiction",
      "author" : "Herman Melville",
      "title" : "Moby Dick",
      "isbn" : "0-553-21311-3",
      "price" : 8.99
   }
]
```



倒数第二本书：$..book[-2]

```json
[
   {
      "category" : "fiction",
      "author" : "Herman Melville",
      "title" : "Moby Dick",
      "isbn" : "0-553-21311-3",
      "price" : 8.99
   }
]
```



前两本书：$..book[0,1]或者$..book[:2]

```json
[
   {
      "category" : "reference",
      "author" : "Nigel Rees",
      "title" : "Sayings of the Century",
      "price" : 8.95
   },
   {
      "category" : "fiction",
      "author" : "Evelyn Waugh",
      "title" : "Sword of Honour",
      "price" : 12.99
   }
]
```



倒数二本书：$..book[-2:]

```json
[
   {
      "category" : "fiction",
      "author" : "Herman Melville",
      "title" : "Moby Dick",
      "isbn" : "0-553-21311-3",
      "price" : 8.99
   },
   {
      "category" : "fiction",
      "author" : "J. R. R. Tolkien",
      "title" : "The Lord of the Rings",
      "isbn" : "0-395-19395-8",
      "price" : 22.99
   }
]
```



第三本书到最后：$..book[2:]

```json
[
   {
      "category" : "fiction",
      "author" : "Herman Melville",
      "title" : "Moby Dick",
      "isbn" : "0-553-21311-3",
      "price" : 8.99
   },
   {
      "category" : "fiction",
      "author" : "J. R. R. Tolkien",
      "title" : "The Lord of the Rings",
      "isbn" : "0-395-19395-8",
      "price" : 22.99
   }
]
```



书中带isbn字段的：$..book[?(@.isbn)]

```json
[
   {
      "category" : "fiction",
      "author" : "Herman Melville",
      "title" : "Moby Dick",
      "isbn" : "0-553-21311-3",
      "price" : 8.99
   },
   {
      "category" : "fiction",
      "author" : "J. R. R. Tolkien",
      "title" : "The Lord of the Rings",
      "isbn" : "0-395-19395-8",
      "price" : 22.99
   }
]
```



书的价格小于10的：$.store.book[?(@.price < 10)]或者$..book[?(@.price <= $['expensive'])]

```json
[
   {
      "category" : "reference",
      "author" : "Nigel Rees",
      "title" : "Sayings of the Century",
      "price" : 8.95
   },
   {
      "category" : "fiction",
      "author" : "Herman Melville",
      "title" : "Moby Dick",
      "isbn" : "0-553-21311-3",
      "price" : 8.99
   }
]
```



作者匹配正则：$..book[?(@.author =~ /.*REES/i)]

```json
[
   {
      "category" : "reference",
      "author" : "Nigel Rees",
      "title" : "Sayings of the Century",
      "price" : 8.95
   }
]
```



书本的数量：$..book.length()

```json
[
   4
]
```



4.JsonPath高级特性：

**(1)JasonPath解析方式：**

通过静态读取API

```java
String json =  “ ... ” ;
List < String > authors =  JsonPath 。read（json，“ $ .store.book [*]。author ”）;
```



多次调用：

```java
String json = "...";
Object document = Configuration.defaultConfiguration().jsonProvider().parse(json);

String author0 = JsonPath.read(document, "$.store.book[0].author");
String author1 = JsonPath.read(document, "$.store.book[1].author");
```



链式获取：

```json
String json = "...";

ReadContext ctx = JsonPath.parse(json);

List<String> authorsOfBooksWithISBN = ctx.read("$.store.book[?(@.isbn)].author");


List<Map<String, Object>> expensiveBooks = JsonPath
                            .using(configuration)
                            .parse(json)
                            .read("$.store.book[?(@.price > 10)]", List.class);
```



**(2)返回数据类型转换**

```java
String author = JsonPath.parse(json).read("$.store.book[0].author")
    
    
String json = "{\"date_as_long\" : 1411455611975}";
Date date = JsonPath.parse(json).read("$['date_as_long']", Date.class);


Book book = JsonPath.parse(json).read("$.store.book[0]", Book.class);


TypeRef<List<String>> typeRef = new TypeRef<List<String>>() {};
List<String> titles = JsonPath.parse(JSON_DOCUMENT).read("$.store.book[*].title", typeRef);
```







**(3).过滤器**

```java
import static com.jayway.jsonpath.JsonPath.parse;
import static com.jayway.jsonpath.Criteria.where;
import static com.jayway.jsonpath.Filter.filter;
...
...

Filter cheapFictionFilter = filter(
   where("category").is("fiction").and("price").lte(10D)
);

List<Map<String, Object>> books =  
   parse(json).read("$.store.book[?]", cheapFictionFilter);
```



```json
[{
	"category": "fiction",
	"author": "Herman Melville",
	"title": "Moby Dick",
	"isbn": "0-553-21311-3",
	"price": 8.99
}]
```



其他：

```json
Filter fooOrBar = filter(
   where("foo").exists(true)).or(where("bar").exists(true)
);
   
Filter fooAndBar = filter(
   where("foo").exists(true)).and(where("bar").exists(true)
);
```



自定义：

```java
Predicate booksWithISBN = new Predicate() {
    @Override
    public boolean apply(PredicateContext ctx) {
        return ctx.item(Map.class).containsKey("isbn");
    }
};

List<Map<String, Object>> books = 
   reader.read("$.store.book[?].isbn", List.class, booksWithISBN);
```

































