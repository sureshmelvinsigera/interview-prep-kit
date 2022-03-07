# Spring Data JPA - Learnings

## Understanding Datasource & Connection

## Key Learnings List

* @Modifying is meaningless without @Query annotation
* @Modifying needs @Transactional. Otherwise will result in TransactionRequiredException
* @Transactional only rollback on RuntimeException
* @Transactional(readOnly = true) can be avoided as readOnly adds few performance overheads unncessarily
* Inside @Transactional, if you get a fetch a Entity & update the Entity, you don't need to explicitly save the Entity

## Key Learnings about the Mapping 

### One to One
* For @OneToOne associations, you should always share the Primary Key with the parent table, and you should avoid the bidirectional association
* PK and FK columns are most often indexed, so sharing the PK can reduce the index footprint by half, which is desirable since you want to store all your indexes into memory to speed up index scanning

```java
@Entity
@Table(name = "post")
public class Post {
 
    @Id
    @GeneratedValue
    private Long id;
 
    private String title;
 
    @OneToOne(mappedBy = "post", cascade = CascadeType.ALL,
              fetch = FetchType.LAZY, optional = false)
    private PostDetails details;
}


@Entity
@Table(name = "post_details")
public class PostDetails {
 
    @Id
    private Long id;
 
    @OneToOne(fetch = FetchType.LAZY)
    @MapsId
    private Post post; 
}
```
**Reference** : https://vladmihalcea.com/the-best-way-to-map-a-onetoone-relationship-with-jpa-and-hibernate/

## Interesting facts

* All fields in @Entity class are default persistable

<hr>

## @Query Annotation

### 1. How to use the method args in @Query

```java
// Access params using param's position
@Query("select u from User u where u.emailAddress = ?1")
User findByEmailAddress(String emailAddress);

// Access params using param's name
@Query("select u from User u where u.firstname = :firstname or u.lastname = :lastname")
User findByLastnameOrFirstname(@Param("lastname") String lastname, @Param("firstname") String firstname);
```

### 2. How to write a update/delete query

```java
@Modifying
@Query("update User u set u.firstname = ?1 where u.lastname = ?2")
int setFixedFirstnameFor(String firstname, String lastname);
```

### 3. Query from Method name is slow for Modifying queries

Consider the following delete methods

```java
public interface EmployeeRepo extends JpaRepository<Employee, Integer>{

    void deleteById(Integer id);

    @Modifying
    @Query("delete from Employee e where e.id = ?1")
    void deleteByGivenId(Integer id);
}
```

```java
// Queries generated 

void deleteById(Integer id);

Hibernate: 
    select
        employee0_.id as id1_0_0_,
        employee0_.name as name2_0_0_ 
    from
        employee employee0_ 
    where
        employee0_.id=?
Hibernate: 
    delete 
    from
        employee 
    where
        id=?

---

@Modifying
@Query("delete from Employee e where e.id = ?1")
void deleteByGivenId(Integer id);
    
Hibernate: 
    delete 
    from
        employee 
    where
        id=?
```

### One to One Ownership

In below example, customer is the owner of the address

```
@Entity
class Customer {

    @OneToOne
    Address address;
}

@Entity
class Address {

    @OneToOne(mappedBy="address");
    Customer customer;
}
```
