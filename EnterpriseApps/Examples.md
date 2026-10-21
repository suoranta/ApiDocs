# Examples: Suoranta.EnterpriseLib

## Entity-DTO-Entity transformations (Suoranta.EnterpriseLib.Util.AutoMapper)
EnterpriseLib simplifies configuring AutoMapper in scenarios that involve graphs of database entities. The key base classes are:
- `EntityProfile` (derives AutoMapper's `Profile` class)
- `DTO<T>` for DTO types. DTO types are not required to be derived, but should always implement `IDataTransferObject`.
- `EntityResolver<TDto, TEntity>`, provides implementations for AutoMapper's `IValueResolver` interface.

Suoranta.WebApp defines two related attributes: 
- `DTOText` should be used to decorate the property that provides the basic text representation for the instance.
- If DTO wraps a singular value, that should be decorated with `DTOValue` attribute.

An example of DTO class is given below. Note that the type parameter is the derived class. This is purely due to performance considerations: retrieving runtime type information for each derived instance is inefficient, and the type parameter circumvents this.
```csharp
public class PersonDTO : DTO<PersonDTO>, IDataTransferObject
{
    #region Entity props
    [DisplayName("First Name")]     public string FirstName { get; set; }
    [DisplayName("Last Name")]      public string LastName { get; set; }
    [DisplayName("Date of Birth")]  public DateTime BirthDate { get; set; }
    [DisplayName("Nationality")]    public string Nationality { get; set; }
    [DisplayName("Address")]        public int AddressId { get; set; }
    #endregion

    [DisplayName("Name"), DTOText]  public string Name { get; set; }
    [Browsable(false)]              public AddressDTO Address { get; set; }
    [DisplayName("Street Address")] public string StreetAddress => Address.Name;

    public override Type EntityType => typeof(Person);
}
```

Updating database entities requires DTO to be fully "reattached" to the original entity graph. To achieve this, create `EntityResolver` for each entity type that has updateable navigation properties. Single derived class can implement all
the needed `IValueResolver`s.
```csharp
public class PersonResolver : EntityResolver<PersonDTO, Person>, IValueResolver<PersonDTO, Person, Address>
{
    public PersonResolver(DbContext dbContext) : base(dbContext)
    { }

    public Address Resolve(PersonDTO source, Person destination, Address destMember, ResolutionContext context)
        => Resolve<Address>(source, destination, destMember, context);
}
```

The AutoMapper `Profile` for the preceding types is as follows:
```csharp
public class Maps : EntityProfile
{
    public Maps()
    {
        CreateMap<Address, AddressDTO>()
            .ForMember(dto => dto.Name, conf => conf.MapFrom(Address.StreetAddress));
        CreateMap<Person, PersonDTO>()
            .ForMember(dto => dto.Name, conf => conf.MapFrom(p => p.FirstName + " " + p.LastName))
            .AutoIgnore();

        // Reverse mappings from DTO to entity
        PersonDTOToPerson();
    }

    IMappingExpression<PersonDTO, Person> PersonDTOToPerson()
    {
        // GetEntity returns DB entity or creates a new one if a matching id is not found in the DB
        return CreateMapToEntity<PersonDTO, Person, PersonResolver>()
            .ForMember(entity => entity.Address, conf => conf.MapFrom<PersonResolver>());
    }
}
```

## DTO-To-LINQ (Suoranta.EnterpriseLib.Linq)
EnterpriseLib's DTO-To-LINQ provides a lightweight, easily customizable alternative to ODATA.

Assume we have a database-backed entity class `Post` and we want to build a form to query entities flexibly. Here's an excerpt from the class definion:
```csharp
public class Post : Entity
{
    public BlogReaderType ReaderType { get; set; }

    // ...

    [NotMapped] public DateTime RangeStart => PublishedDate;

    [NotMapped] public DateTime RangeEnd => ExpirationDate;
}
```

```csharp
public enum BlogReaderType
{
    Minor            = 1,
    Adult            = 2,
    PartyMember      = 4,
    SecurityPolice   = 8,
    // ...
}
```
The full enum has additional fields decorated with `Mask` attribute to indicate that they are not primary settable values but define multi-value filtration options. Advanced modeling features such as this are an essential quality of EnterpriseLib, but we leave this part out of this basic example.

Data transfer objects (DTOs) that represent a query translateable to LINQ derive from `EntityQuery<T>`, where the type parameter represents the queryable entity type.
```csharp
public class PostQueryFormData : EntityQuery<Post>
{
    [ValueFilter(nameof(Post.ReaderType), Comparison.Equal)]
    public ValueFilter<int> CensorshipLevel { get; set; }

    [RangeFilter(nameof(Post.RangeStart), nameof(Post.RangeEnd), FilterMode = RangeFilterMode.OverlapsRange)]
    public RangeFilter<DateTime> DateRange { get; set; }
}
```
As we can see, the interpretation of query parameters is defined by decorating them with attributes. Names of the entity properties, given to the attribute as parameter, can be navigation properties. The query is constructed by calling its `ApplyAllFilters` method. This does not enumerate the results, hence custom LINQ can be appended before the query is actually executed against the database.

Built-in filters are defined in namespace `Suoranta.EnterpriseLib.Linq.Filters`.

### StringSetFilter
Many (legacy) databases contain string (char or varchar) columns that represent a value from a fixed set of choices. Furthermore, the data might not be fully normalized: the typical issue is variation in character casing. EnterpriseLib provides the specialized base class `StringSetFilter` to ensure robust querying of these columns. The filter is used by creating a derived class:
```csharp
public class CountryFilter : StringSetFilter
{
    public CountryFilter(params string[] values) : base(values)
    { }

    public const string Finland = "FINLAND";
    public const string Sweden = "SWEDEN";
    public const string UK = "United Kingdom";
    public const string USA = "UNITED STATES";
}
```
The set of all known database values can be defined in two ways:
- Defining them as constant fields of the class, as seen above.
- By overriding the `GetAllValues` method.

The filter is included in a DTO as follows:
```csharp
[StringSetFilter("Address.Country")]
public CountryFilter Country { get; }
```
In the example above, the target property is a navigation property of Address entity.
