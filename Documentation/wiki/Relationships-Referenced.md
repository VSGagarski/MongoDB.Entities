# Referenced Relationships

referenced relationships require a bit of special handling. a **one-to-one** relationship is defined by using the `One<T>` class and **one-to-many** as well as **many-to-many** relationships are defined by using the `Many<T>` class. it is also a good idea to initialize the `Many<T>` properties in the constructor of the parent entity as shown below in order to avoid null-reference exceptions during runtime.
```csharp
public class Book : Entity
{
    public One<Author> MainAuthor { get; set; }
    public Many<Author> Authors { get; set; }

    [OwnerSide]
    public Many<Genre> AllGenres { get; set; }

    public Book()
    {
        this.InitOneToMany(() => Authors);
        this.InitManyToMany(() => AllGenres, genre => genre.AllBooks);
    }
}
```
notice the parameters of the `InitOneToMany` and `InitManyToMany` methods above. the first method only takes one parameter which is just a lambda pointing to the property you want to initialize.

the next method takes 2 parameters. first is the property to initialize. second is the property of the other side of the relationship.

also note that you specify which side of the relationship a property is by using the attributes `[OwnerSide]` or `[InverseSide]` for defining many-to-many relationsips.

## One-to-one

call the `ToReference()` method of the entity you want to store as a reference like so:

```csharp
book.MainAuthor = author.ToReference();
await book.SaveAsync();
```
alternatively you can use the implicit operator functionality by simply assigning an instance or the string ID like so:
```csharp
book.MainAuthor = author;
book.MainAuthor = author.ID;
```

### Reference removal
```csharp
book.MainAuthor = null;
await book.SaveAsync();
```
the original `author` in the `Authors` collection is unaffected.

### Entity deletion
If you delete an entity that is referenced as above by calling `author.DeleteAsync()` all references pointing to that entity are automatically deleted. as such, `book.MainAuthor.ToEntityAsync()` will then return `null`. the `.ToEntityAsync()` method is described below.

## One-to-many & many-to-many
```csharp
await book.Authors.AddAsync(author); //one-to-many
await book.AllGenres.AddAsync(genre); //many-to-many
```
there's no need to call `book.SaveAsync()` because references are automatically saved using special join collections. you can read more about them in the [Schema Changes](Schema-Changes.md) section.

### Reference removal
```csharp
await book.Authors.RemoveAsync(author);
await .AllGenres.RemoveAsync(genre);
```

the original `author` in the `Authors` collection is unaffected. also the `genre` entity in the `Genres` collection is unaffected. only the relationship between entities are deleted.

### Entity deletion
If you delete an entity that is referenced as above by calling `author.DeleteAsync()` all references pointing to that `author` entity are automatically deleted. as such, `book.Authors` will not have `author` as a child. the same applies to `Many-To-Many` relationships. deleting any entity that has references pointing to it from other entities results in those references getting deleted and the relationships being invalidated.

# ToEntityAsync() shortcut

a reference can be turned back in to an entity with the `ToEntityAsync()` method.

```csharp
var author = await book.MainAuthor.ToEntityAsync();
```
you can also project the properties you need instead of getting back the complete entity like so:
```csharp
var author = await book.MainAuthor
                       .ToEntityAsync(a => new Author
                        {
                          Name = a.Name,
                          Age = a.Age
                        });
```

# Transaction support
adding and removing related entities require passing in the session when used within a transaction. see [here](Transactions.md) for an example.
