# sORM
Simple ORM and nothing more.

### Mappings
To link class and table in database you need to decorate it with `DataModel` attribute.
Then you need to decorate fields you want to see in database with `MapAsType` attributes and decorate key field (Id or etc.) with `Key` attribute.
You can also use `MapAuto` attribute to let the ORM detect and map type.

Supported types for auto mapping:

* `string` as **VARCHAR(MAX)**
* `int` as **INT**
* `bool` as **BIT**
* `float` as **REAL**
* `Guid` as **UNIQUEIDENTIFIER**

Example: 
```c#
  [DataModel]
  class MyClass
  {
    [MapAuto]
    public int MyProperty { get; set; }
  
    [MapAsType(DataType.String)]
    public string MyProperty2 { get; set; }
  
    [MapAsType(DataType.Guid)]
    [Key]
    public Guid Id { get; set; }
  }
```

### Connection setup
On startup you need to setup connection string:

```c#
  SimpleORM.Current.Initialize(@"%CONNECTION_STRING_EXAMPLE%");
```

### Creating and updating objects
To create or update records use `CreateOrUpdate` method:

```c#
  var obj = new MyClass() { MyProperty = 1, MyProperty2 = "foo" };
  SimpleORM.Current.CreateOrUpdate(obj);
  obj.MyProperty = 666;
  SimpleORM.Current.CreateOrUpdate(obj);
```

### Deleting objects
To delete exact object use `Delete` method:

```c#
  SimpleORM.Current.Delete(obj);
```

### Conditions
You can get or delete bunch of records using **Conditions**.
**Conditions** is a way to create complex conditions using *Equals*, *NotEquals*, *More*, *Less* and *Like* operators.
**Conditions** can be modified using *And* and *Or* operators.

Example:

```c#
  SimpleORM.Current.Delete<MyClass>(
    Operator.Or(Condition.Equals("MyProperty2", "foo"), Condition.Equals("MyProperty", 2))
  );
  
  SimpleORM.Current.Delete<MyClass>(
    Operator.And(
      Condition.Equals("MyProperty2", "123"), 
      Operator.Or(
        Condition.Equals("MyProperty2", "333"), Condition.Equals("MyProperty", 999)
      )
    )
  );

  var result = SimpleORM.Current.Get<MyClass>(
    Condition.Equals("MyProperty2", "baz"),
    new DataEntityListLoadOptions(size: 1, index: 2)
  );
```

### References

To create one-to-many reference from one entity to another you need to use `Key` or `SecondaryKey` attribute on property that will be referenced, and `ReferenceTo` attribute on property that will be referencing.

*Note that properties decorated with `Key` and `SecondaryKey` will be NOT NULL and UNIQUE*

Example:
```c#
  [DataModel]
  class A
  {
    [MapAuto]
    [Key]
    public Guid Id { get; set; }
  }
  
  [DataModel]
  class B
  {
    [MapAuto]
    [Key]
    public Guid Id { get; set; }
    
    [MapAuto]
    [SecondaryKey]
    public int MyIntValue { get; set; }
  }
  
  [DataModel]
  class C
  {
    [MapAuto]
    [Key]
    public Guid Id { get; set; }
    
    [MapAuto]
    [ReferenceTo(typeof(A), "Id"]
    public Guid AnotherId { get; set; }
    
    [MapAuto]
    [ReferenceTo(typeof(B), "MyIntValue"]
    public int MyIntValueReference { get; set; }
  }
```

### Extras
To get SQL what has been executed (for logging etc.) you should use `AddOnRequestListener`.

Example:
```c#
  var logger = AwesomeFramework.Get<IAwesomeLoggingTool>(); //example
  SimpleORM.Current.AddOnRequestListener( (sql) => logger.Log("Executed SQL: " + sql) );
```
