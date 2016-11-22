# EntityFramework.BulkExtensions

   This project was built as an extension to add bulk operations functionality to the Entity Framework. 
It works as extension methods of the DBContext class and is very simple to use. It supports transaction if the context's database have a CurrentTransaction, or it creates an internal one for the scope of the operation.
<br><br>
   It relies on the SqlBulkCopy class to perform all the operations, because of that, it can't handle navigation properties and will not persist relationships between entities, but there is a workaround for that if the foreign keys are being explicitly mapped in your model classes. See the workaround in the examples below.
##How to use it

###Bulk insert
   There is two ways of using this method. By only using the list as parameters for this extension method it will perform a standard SqlBulkCopy operation, witch will not return the Ids of the inserted entities because of a limitation of the SqlBulkCopy class. 
   <br><br>
   By also selecting Identity.InputOutput as the second parameter, the method will fill the generated Ids for the entities inserted, using temporary tables to output and select the generated Ids under the hood. See the exemples below:
```c#
using EntityFramework.BulkExtensions.Operations

var entityList = new List<MyEntity>();

entityList.Add(new MyEntity());
entityList.Add(new MyEntity());
entityList.Add(new MyEntity());

//Bulk insert extension method
context.BulkInsert(entityList); 

/* Also, if you want the generated ids you can use the code below */

context.BulkInsert(entityList, Identity.InputOutput);
entityList.First().Id //would return the id generated on the insert.

/* The ids generated by the database will be set for every inserted item
   in the entities collection */
```

####Workaround for relationships
   You can explicitly set the foreign keys of your entity and insert it. See the example below.
   
```c#
using EntityFramework.BulkExtensions.Operations

var role = context.Set<Roles>()
   .Single(entity => entity.Name == "Admin")
   .ToList();

var entityList = new List<User>();

entityList.Add(new User{ RoleId = role.Id }); //Set the role id on the newly created user
entityList.Add(new User{ RoleId = role.Id });
entityList.Add(new User{ RoleId = role.Id });
entityList.Add(new User{ RoleId = role.Id });
entityList.Add(new User{ RoleId = role.Id });
entityList.Add(new User{ RoleId = role.Id });

//Bulk insert extension method
context.BulkInsert(entityList); 
/* By explicitly setting the foreing key the relationship will be persisted in the database. */
```
   
###Bulk update
```c#
using EntityFramework.BulkExtensions.Operations

Random rnd = new Random();

//Read some entities from database.
var entityList = context.Set<MyEntity>()
   .Where(entity => entity.Owner == "Steve")
   .ToList();
foreach(var entity in entityList) 
{
    //Replace the old value with some random new value.
    entity.Value = rnd.Next(1000); 
}

//Bulk update extension method
context.BulkUpdate(entityList); 

/* Under the hood, this operation will create a mirror table of your entity's table, 
   bulk insert the updated entities using the SqlBulkCopy class, use the MERGE sql 
   command to transfer the data to the original entity table using the primary keys 
   to match entries and then drop the mirror table. The original course of action of 
   the entity framework would be create an UPDATE command for each entity, wich suffers 
   a big performance hit with an increased number of entries to update. */
```

###Bulk delete
```c#
using EntityFramework.BulkExtensions.Operations

//Read some entities from database.
var entityList = context.Set<MyEntity>()
      .Where(entity => entity.Owner == "Steve")
      .toList();

//Bulk delete extension method
context.BulkDelete(entityList); 

/* This operation will delete all the entities in the list from the database. */

