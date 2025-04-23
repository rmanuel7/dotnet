# [IValidatableObject](https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation?view=aspnetcore-9.0#ivalidatableobject)

The preceding example works only with `Movie` types. Another option for class-level validation is to implement [IValidatableObject](https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.ivalidatableobject) in the model class, as shown in the following example:

```C#
public class ValidatableMovie : IValidatableObject
{
    private const int _classicYear = 1960;

    public int Id { get; set; }

    [Required]
    [StringLength(100)]
    public string Title { get; set; } = null!;

    [DataType(DataType.Date)]
    [Display(Name = "Release Date")]
    public DateTime ReleaseDate { get; set; }

    [Required]
    [StringLength(1000)]
    public string Description { get; set; } = null!;

    [Range(0, 999.99)]
    public decimal Price { get; set; }

    public Genre Genre { get; set; }

    public bool Preorder { get; set; }

    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        if (Genre == Genre.Classic && ReleaseDate.Year > _classicYear)
        {
            yield return new ValidationResult(
                $"Classic movies must have a release year no later than {_classicYear}.",
                new[] { nameof(ReleaseDate) });
        }
    }
}
```

```C#
// dataannotations validation c#
// https://www.c-sharpcorner.com/article/validations-forms-in-dataannotations/
// https://weblogs.asp.net/ricardoperes/net-8-data-annotations-validation

public static void TryValidateObjectExample1()  
{  
    /// 1.- Create a customer  
    var customer = new Customer  
    {  
        Name                 = string.Empty,  
        EntryDate            = DateTime.Today,  
        Password             = "AAAA",  
        PasswordConfirmation = "BBBB",  
        Age                  = -1  
    };  
    /// 2.- Create a context of validation  
    ValidationContext valContext = new ValidationContext(customer, null, null);  
    /// 3.- Create a container of results  
    var validationsResults = new List<ValidationResult>();  
    /// 4.- Validate customer  
    bool correct = Validator.TryValidateObject(customer, valContext, validationsResults, true);  
   
    Console.WriteLine(validationsResults.ToDescErrorsString("Without Errors !!!!"));  
    Console.Read();  
} 
```
