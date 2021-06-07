---
layout: post
title: "Introduction to Spring Data JDBC"
date: 2020-02-27 10:04
tags: [spring, spring-data, jdbc, how-to]
summary: An introductory guide to getting started with Spring Data JDBC
---

In 2018, Spring Data JDBC was [announced](https://spring.io/blog/2018/09/17/introducing-spring-data-jdbc). The purpose was to provide developers with a simpler alternative to JPA while still following the Spring Data paradigm. If you'd like to know more about the motivations behind the project, check out the [reference documentation](https://docs.spring.io/spring-data/jdbc/docs/current/reference/html/#jdbc.why).

In this guide I'll be stepping through some common use cases for Spring Data JDBC. It won't be an in-depth guide, but hopefully give you enough of an introduction to try it yourself. This guide will be most beneficial for those with working knowledge of Spring Data JPA. As usual, you can follow along with the source code over on [github](https://github.com/lumberjackdev/getting-started-spring-data-jdbc).

_Also see the [template](https://github.com/lumberjackdev/spring-boot-java-11-postgres-template) I used for this example if you'd like to get started quickly_

### Getting Started
For our dependencies we'll just be using data-jdbc starter, flyway to manage the schema, and the postgres driver to connect to the database.
```groovy
// build.gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jdbc'

    implementation 'org.flywaydb:flyway-core'

    runtimeOnly 'org.postgresql:postgresql'
}
```

Next we'll need to configure the application to connect to the database:
```yaml
# application.yml
spring:
  application:
    name: template-app
  datasource:
    url: jdbc:postgresql://localhost:5432/demo_app?currentSchema=app
    username: app_user
    password: change_me
    driver-class-name: org.postgresql.Driver
```

### Entity Mapping
Now that we have the app connected to the database, let's take a look at a sample class. For this example, we'll be  using this sql table:

```sql
create table book (
    id varchar(32) not null,
    title varchar(255) not null,
    author varchar(255),
    isbn varchar(15),
    published_date date,
    page_count integer,
    primary key (id)
);
```

And so we have our corresponding java class (note that `@Id` is imported from `org.springframework.data.annotation.Id`):
```java
// Book.java
public class Book {
    @Id
    private String id;
    private String title;
    private String author;
    private String isbn;
    private Instant publishedDate;
    private Integer pageCount;
}
```

However, running the [test](https://github.com/lumberjackdev/getting-started-spring-data-jdbc/blob/master/src/test/java/com/lumberjackdev/jdbcexample/repository/BookRepositoryTest.java):
```java
// BookRepositoryTest.java
@Test
void canSaveBook() {
    var book = Book.builder().author("Steven Erikson").title("Gardens of the Moon").build();
    var savedBook = bookRepository.save(book);

    assertThat(savedBook.getId()).isNotBlank();
    assertThat(savedBook.getAuthor()).isEqualTo(book.getAuthor());
    assertThat(savedBook.getTitle()).isEqualTo(book.getTitle());

    assertThat(savedBook).isEqualTo(bookRepository.findById(savedBook.getId()).get());
}
```
We'll see an error that looks like `ERROR: null value in column "id" violates not-null constraint`. You'll see this if you haven't defined a way for ids to be generated or defined a default value in the class. In Spring Data JDBC this looks [little different](https://docs.spring.io/spring-data/jdbc/docs/current/reference/html/#jdbc.entity-persistence.id-generation) from what you may have seen in Spring Data JPA. In our case we'll need to define an `ApplicationListener` for a `BeforeSaveEvent`:

```java
// PersistenceConfig.java
@Bean
public ApplicationListener<BeforeSaveEvent> idGenerator() {
    return event -> {
        var entity = event.getEntity();
        if (entity instanceof Book) {
            ((Book) entity).setId(UUID.randomUUID().toString());
        }
    };
}
```

And with that our test will pass because the Id field will now be set. For a full list of supported life cycle events check out the [docs](https://docs.spring.io/spring-data/jdbc/docs/current/reference/html/#jdbc.events).

### Query Methods
One of the features that the Spring Data modules have in common is the ability to define custom Query methods for repositories. Spring data JDBC take a slightly different approach to this. To see this in action we'll define a query method for our `BookRepository`:

```java
Optional<Book> findByTitle(String title);
```

And if we run the associated test:

```java
@Test
void canFindBookByTitle() {
    var title = "Gardens of the Moon";
    var book = Book.builder().author("Steven Erikson").title(title).build();
    var savedBook = bookRepository.save(book);
    assertThat(bookRepository.findByTitle(title).get()).isEqualTo(savedBook);
}
```

We get a stacktrace with an error that looks something like: `Caused by: java.lang.IllegalStateException: No query specified on findByTitle`. Currently Spring Data JDBC only [supports](https://docs.spring.io/spring-data/jdbc/docs/current/reference/html/#jdbc.query-methods.at-query) queries with explicitly declared `@Query` definitions. So we can update our query method:

```java
@Query("select * from Book b where b.title = :title")
Optional<Book> findByTitle(@Param("title") String title);
```
And now the test passes! You'll need to keep this in mind when defining custom repository methods. 

### Relationships
Like query methods, Spring Data JDBC also takes a different approach to relationships. Mainly that there is no lazy loading, so if you ever don't want a relationship on an Entity, just leave it off of the class. This comes from the concept that in Domain Driven Design our entities that we'll be fetching are aggregate roots, so by design aggregates should be pulling back other classes.

#### One-to-One
For One-to-One and One-to-Many relationships, we'll be using the same `@MappedCollection` annotation. First, we'll look at One-to-One relationships. In this case, there will be a `UserAccount` object with a reference to an `Address`. Here's the associated sql:

```sql
create table address
(
    id      varchar(36) not null,
    city    varchar(255),
    state   varchar(255),
    street  varchar(255),
    zipcode varchar(255),
    primary key (id)
);
```

And the user_account table:

```sql
create table user_account
(
    id         varchar(36)  not null,
    name       varchar(255) not null,
    email      varchar(255) not null,
    address_id varchar(36),
    primary key (id),
    constraint fk_user_account_address_id foreign key (address_id) references address (id)
);
```

The `UserAccount` class looks something like this:

```java
// UserAccount.java
public class UserAccount implements GeneratedId {
    // ...other fields
    @MappedCollection(idColumn = "id")
    private Address address;
}
```
While the other fields are left out, what's shown is the important mapping to the `Address` class. In this case `idColumn` is the id field name in the `Address` class. Note that the `Address` class has no reference to the `UserAccount` class since `UserAccount` is the aggregate. This is demonstrated in the test:

```java
//UserAccountRepositoryTest.java
@Test
void canSaveUserWithAddress() {
    var address = stubAddress();
    var newUser = stubUser(address);

    var savedUser = userAccountRepository.save(newUser);

    assertThat(savedUser.getId()).isNotBlank();
    assertThat(savedUser.getAddress().getId()).isNotBlank();

    var foundUser = userAccountRepository.findById(savedUser.getId()).orElseThrow(IllegalStateException::new);
    var foundAddress = addressRepository.findById(foundUser.getAddress().getId()).orElseThrow(IllegalStateException::new);

    assertThat(foundUser).isEqualTo(savedUser);
    assertThat(foundAddress).isEqualTo(savedUser.getAddress());
}
```

#### One-to-Many
Here's the sql we'll be using to showcase a One-to-Many relationship:

```sql
create table warehouse
(
    id       varchar(36) not null,
    location varchar(255),
    primary key (id)
);
```

And the inventory_item table:

```sql
create table inventory_item
(
    id        varchar(36) not null,
    name      varchar(255),
    count     integer,
    warehouse varchar(36),
    primary key (id),
    constraint fk_inventory_item_warehouse_id foreign key (warehouse) references warehouse (id)
);
```

In this example the `warehouse` has many `inventory_items`. So for the associated `Warehouse` class, we'll use `@MappedCollection` again to reference `InventoryItem`:

```java
public class Warehouse {
    // ...other fields
    @MappedCollection
    Set<InventoryItem> inventoryItems = new HashSet<>();

    public void addInventoryItem(InventoryItem inventoryItem) {
        var itemWithId = inventoryItem.toBuilder().id(UUID.randomUUID().toString()).build();
        this.inventoryItems.add(itemWithId);
    }
}

public class InventoryItem {
    @Id
    private String id;
    private String name;
    private int count;
}
```
In this example we're setting the `Id` field in a helper method for adding items to the `Warehouse`. We could also define an `ApplicationListener` for a `BeforeSaveEvent` specifically for the `Warehouse` class that sets the Id field for every `InventoryItem`, we don't have to do it the way I did here. Check out the [tests](https://github.com/lumberjackdev/getting-started-spring-data-jdbc/blob/master/src/test/java/com/lumberjackdev/jdbcexample/repository/WarehouseRepositoryTest.java) to see some of the behavior for One-to-Many relationships. The main thing to notice is that when we save or delete an instance of `Warehouse` then the corresponding `InventoryItems` are also affected. 

In our case, we have no need for `InventoryItem` to know about the `Warehouse`. So this class has only the fields that exist to describe it. In JPA it's common to build out both sides of a relationship, but this can get a little cumbersome and error prone if we forget to maintain both sides. Spring Data JDBC encourages only setting the sides of relationships that we need, so the corresponding Many-to-One mapping is left off in this case. 

#### Many-to-One and Many-to-Many
For the purposes of this guide, I won't be going into any detail on Many-to-One or Many-to-Many relationships. My advice for Many-to-Many relationships is usually to avoid them unless absolutely necessary - they can be unavoidable sometimes. Both of these relationship types are possible in Spring Data JDBC by referencing only the Id of the related entities. So know that there's a little more work involved. As I learn more about Spring Data JDBC, I'll publish another post to discuss these relationships. 

### Final Thoughts
Most of what I've covered should feel pretty familiar if you've used Spring Data JPA. I did mention earlier that Spring Data JDBC aims to be simpler. I mentioned that there's no lazy loading on relationships, and beyond this simplicity is achieved by eliminating caching, dirty tracking, and sessions all together. In Spring Data JDBC if we load an entity, it's fully loaded (relationships included) and it's only saved when we save it to the repository. The examples I showed my appear almost identical to their JPA counterparts, but know that these concepts don't exist in this module. 

Overall, I like Spring Data JDBC. I admit that it may not be the first choice for every application, however, I would encourage giving it a try. As someone who's struggled with things like lazy loading and dirty tracking in the past I appreciate the straightforward nature it takes. I think it could be my go to choice for simpler domains that don't require a lot of custom queries. 

That's all for now, thanks for reading! Hopefully you found this guide useful and it gives you a starting point for using Spring Data JDBC.
