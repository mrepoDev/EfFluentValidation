<!--
GENERATED FILE - DO NOT EDIT
This file was generated by [MarkdownSnippets](https://github.com/SimonCropp/MarkdownSnippets).
Source File: /readme.source.md
To change this file edit the source file and then run MarkdownSnippets.
-->

# <img src="/src/icon.png" height="30px"> FluentValidation.EntityFramework

[![Build status](https://ci.appveyor.com/api/projects/status/lqms26lthr90jhva?svg=true)](https://ci.appveyor.com/project/SimonCropp/fluentvalidation-entityframework)
[![NuGet Status](https://img.shields.io/nuget/v/FluentValidation.EntityFramework.svg)](https://www.nuget.org/packages/FluentValidation.EntityFramework/)

Adds [FluentValidation](https://fluentvalidation.net/) support to [EntityFramework](https://docs.microsoft.com/en-us/ef/core/).

Support is available via a [Tidelift Subscription](https://tidelift.com/subscription/pkg/nuget-fluentvalidation.entityframework?utm_source=nuget-fluentvalidation.entityframework&utm_medium=referral&utm_campaign=enterprise).

<!-- toc -->
## Contents

  * [Usage](#usage)
    * [Define Validators](#define-validators)
    * [Context](#context)
    * [ValidationFinder](#validationfinder)
  * [DbContextValidator](#dbcontextvalidator)
    * [TryValidate](#tryvalidate)
    * [Validate](#validate)
    * [ValidatorTypeCache](#validatortypecache)
    * [ValidatorFactory](#validatorfactory)
  * [DbContext](#dbcontext)
    * [ValidatingDbContext](#validatingdbcontext)
    * [DbContext as a base](#dbcontext-as-a-base)
  * [Security contact information](#security-contact-information)<!-- endtoc -->


## NuGet package

 * https://nuget.org/packages/FluentValidation.EntityFramework/


## Usage


### Define Validators

<!-- snippet: Employee.cs -->
<a id='snippet-Employee.cs'/></a>
```cs
using System.ComponentModel.DataAnnotations.Schema;
using FluentValidation;

public class Employee
{
    [DatabaseGenerated(DatabaseGeneratedOption.None)]
    public int Id { get; set; }

    public int CompanyId { get; set; }
    public Company Company { get; set; } = null!;
    public string? Content { get; set; }
    public int Age { get; set; }

    public class Validator :
        AbstractValidator<Employee>
    {
        public Validator()
        {
            RuleFor(_ => _.Content)
                .NotEmpty();
        }
    }
}
```
<sup><a href='/src/Tests/Snippets/DataContext/Employee.cs#L1-L23' title='File snippet `Employee.cs` was extracted from'>snippet source</a> | <a href='#snippet-Employee.cs' title='Navigate to start of snippet `Employee.cs`'>anchor</a></sup>
<!-- endsnippet -->

See [Creating your first validator](https://docs.fluentvalidation.net/en/latest/start.html).


### Context

Extra context is passed through FluentValidations [CustomContext](https://docs.fluentvalidation.net/en/latest/custom-validators.html#writing-a-custom-validator).

Data:

<!-- snippet: EfContext -->
<a id='snippet-efcontext'/></a>
```cs
public class EfContext
{
    public DbContext DbContext { get; }
    public EntityEntry EntityEntry { get; }
```
<sup><a href='/src/FluentValidation.EntityFramework/Model/EfContext.cs#L6-L13' title='File snippet `efcontext` was extracted from'>snippet source</a> | <a href='#snippet-efcontext' title='Navigate to start of snippet `efcontext`'>anchor</a></sup>
<!-- endsnippet -->

Usage:

<!-- snippet: ValidatorWithContext.cs -->
<a id='snippet-ValidatorWithContext.cs'/></a>
```cs
using System.Diagnostics;
using EfFluentValidation;
using FluentValidation;

public class ValidatorWithContext :
    AbstractValidator<Employee>
{
    public ValidatorWithContext()
    {
        RuleFor(_ => _.Content)
            .Custom((propertyValue, validationContext) =>
            {
                var efContext = validationContext.EfContext();
                Debug.Assert(efContext.DbContext != null);
                Debug.Assert(efContext.EntityEntry != null);

                if (propertyValue == "BadValue")
                {
                    validationContext.AddFailure("BadValue");
                }
            });
    }
}
```
<sup><a href='/src/Tests/Snippets/DataContext/ValidatorWithContext.cs#L1-L23' title='File snippet `ValidatorWithContext.cs` was extracted from'>snippet source</a> | <a href='#snippet-ValidatorWithContext.cs' title='Navigate to start of snippet `ValidatorWithContext.cs`'>anchor</a></sup>
<!-- endsnippet -->


### ValidationFinder

ValidationFinder wraps `FluentValidation.AssemblyScanner.FindValidatorsInAssembly` to provide convenience methods for scanning Assemblies for validators.

<!-- snippet: FromAssemblyContaining -->
<a id='snippet-fromassemblycontaining'/></a>
```cs
var scanResults = ValidationFinder.FromAssemblyContaining<SampleDbContext>();
```
<sup><a href='/src/Tests/Tests.cs#L45-L47' title='File snippet `fromassemblycontaining` was extracted from'>snippet source</a> | <a href='#snippet-fromassemblycontaining' title='Navigate to start of snippet `fromassemblycontaining`'>anchor</a></sup>
<!-- endsnippet -->


## DbContextValidator

`DbContextValidator` performs the validation a DbContext. It has two method:


### TryValidate

<!-- snippet: TryValidateSignature -->
<a id='snippet-tryvalidatesignature'/></a>
```cs
/// <summary>
/// Validates a <see cref="DbContext"/> an relies on the caller to handle those results.
/// </summary>
/// <param name="dbContext">
/// The <see cref="DbContext"/> to validate.
/// </param>
/// <param name="validatorFactory">
/// A factory that accepts a entity type and returns
/// a list of corresponding <see cref="IValidator"/>.
/// </param>
public static async Task<(bool isValid, IReadOnlyList<EntityValidationFailure> failures)> TryValidate(
        DbContext dbContext,
        Func<Type, IEnumerable<IValidator>> validatorFactory)
```
<sup><a href='/src/FluentValidation.EntityFramework/DbContextValidator.cs#L13-L29' title='File snippet `tryvalidatesignature` was extracted from'>snippet source</a> | <a href='#snippet-tryvalidatesignature' title='Navigate to start of snippet `tryvalidatesignature`'>anchor</a></sup>
<!-- endsnippet -->


### Validate

<!-- snippet: ValidateSignature -->
<a id='snippet-validatesignature'/></a>
```cs
/// <summary>
/// Validates a <see cref="DbContext"/> and throws a <see cref="MessageValidationException"/>
/// if any changed entity is not valid.
/// </summary>
/// <param name="dbContext">
/// The <see cref="DbContext"/> to validate.
/// </param>
/// <param name="validatorFactory">
/// A factory that accepts a entity type and returns a
/// list of corresponding <see cref="IValidator"/>.
/// </param>
public static async Task Validate(
        DbContext dbContext,
        Func<Type, IEnumerable<IValidator>> validatorFactory)
```
<sup><a href='/src/FluentValidation.EntityFramework/DbContextValidator.cs#L71-L88' title='File snippet `validatesignature` was extracted from'>snippet source</a> | <a href='#snippet-validatesignature' title='Navigate to start of snippet `validatesignature`'>anchor</a></sup>
<!-- endsnippet -->


### ValidatorTypeCache

`ValidatorTypeCache` creates and caches `IValidator` instances against their corresponding entity type.

It can only be used against validators that have a public default constructor (i.e. no parameters).

<!-- snippet: ValidatorTypeCacheUsage -->
<a id='snippet-validatortypecacheusage'/></a>
```cs
var scanResults = ValidationFinder.FromAssemblyContaining<SampleDbContext>();
var typeCache = new ValidatorTypeCache(scanResults);
var validatorsFound = typeCache.TryGetValidators(typeof(Employee), out var validators);
```
<sup><a href='/src/Tests/Tests.cs#L54-L58' title='File snippet `validatortypecacheusage` was extracted from'>snippet source</a> | <a href='#snippet-validatortypecacheusage' title='Navigate to start of snippet `validatortypecacheusage`'>anchor</a></sup>
<!-- endsnippet -->


### ValidatorFactory

Many APIs take a validation factory with the signature `Func<Type, IEnumerable<IValidator>>` where `Type` is the entity type and `IEnumerable<IValidator>` is all validators for that entity type.

This approach allows a flexible approach on how Validators can be instantiated.


#### DefaultValidatorFactory

`DefaultValidatorFactory` combines [ValidatorTypeCache](#ValidatorTypeCache) and [ValidationFinder](#ValidationFinder).

It assumes that all validators for a DbContext exist in the same assembly as the DbContext and have public default constructors.

Implementation:

<!-- snippet: DefaultValidatorFactory.cs -->
<a id='snippet-DefaultValidatorFactory.cs'/></a>
```cs
using System;
using System.Collections.Generic;
using System.Linq;
using FluentValidation;
using Microsoft.EntityFrameworkCore;

namespace EfFluentValidation
{
    public static class DefaultValidatorFactory<T>
        where T : DbContext
    {
        public static Func<Type, IEnumerable<IValidator>> Factory { get; }

        static DefaultValidatorFactory()
        {
            var validators = ValidationFinder.FromAssemblyContaining<T>();

            var typeCache = new ValidatorTypeCache(validators);
            Factory = type =>
            {
                if (typeCache.TryGetValidators(type, out var enumerable))
                {
                    return enumerable;
                }

                return Enumerable.Empty<IValidator>();
            };
        }
    }
}
```
<sup><a href='/src/FluentValidation.EntityFramework/DefaultValidatorFactory.cs#L1-L30' title='File snippet `DefaultValidatorFactory.cs` was extracted from'>snippet source</a> | <a href='#snippet-DefaultValidatorFactory.cs' title='Navigate to start of snippet `DefaultValidatorFactory.cs`'>anchor</a></sup>
<!-- endsnippet -->


## DbContext

There are several approaches to adding validation to a DbContext


### ValidatingDbContext

`ValidatingDbContext` provides a base class with validation already implemented in `SaveChnages` and `SaveChangesAsync`

<!-- snippet: SampleDbContext.cs -->
<a id='snippet-SampleDbContext.cs'/></a>
```cs
using EfFluentValidation;
using Microsoft.EntityFrameworkCore;

public class SampleDbContext :
    ValidatingDbContext
{
    public DbSet<Employee> Employees { get; set; } = null!;
    public DbSet<Company> Companies { get; set; } = null!;

    public SampleDbContext(DbContextOptions options) :
        base(
            options,
            DefaultValidatorFactory<SampleDbContext>.Factory)
    {
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Company>()
            .HasMany(c => c.Employees)
            .WithOne(e => e.Company)
            .IsRequired();
        modelBuilder.Entity<Employee>();
    }
}
```
<sup><a href='/src/Tests/Snippets/DataContext/SampleDbContext.cs#L1-L25' title='File snippet `SampleDbContext.cs` was extracted from'>snippet source</a> | <a href='#snippet-SampleDbContext.cs' title='Navigate to start of snippet `SampleDbContext.cs`'>anchor</a></sup>
<!-- endsnippet -->


### DbContext as a base

In some scenarios it may not be possible to use a custom base class, I thise case `SaveChnages` and `SaveChangesAsync` can be overridden.

<!-- snippet: CustomDbContext -->
<a id='snippet-customdbcontext'/></a>
```cs
public class SampleDbContext :
    DbContext
{
    public DbSet<Employee> Employees { get; set; } = null!;
    public DbSet<Company> Companies { get; set; } = null!;
    private static Func<Type, IEnumerable<IValidator>> validatorFactory;

    static SampleDbContext()
    {
        validatorFactory = DefaultValidatorFactory<SampleDbContext>.Factory;
    }

    public SampleDbContext(DbContextOptions options) :
        base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Company>()
            .HasMany(c => c.Employees)
            .WithOne(e => e.Company)
            .IsRequired();
        modelBuilder.Entity<Employee>();
    }

    public override int SaveChanges(bool acceptAllChangesOnSuccess)
    {
        DbContextValidator.Validate(this, validatorFactory).GetAwaiter().GetResult();
        return base.SaveChanges(acceptAllChangesOnSuccess);
    }

    public override async Task<int> SaveChangesAsync(
        bool acceptAllChangesOnSuccess,
        CancellationToken cancellationToken = default)
    {
        await DbContextValidator.Validate(this, validatorFactory);
        return await base.SaveChangesAsync(acceptAllChangesOnSuccess, cancellationToken);
    }
}
```
<sup><a href='/src/Tests/Snippets/DataContext/CustomDbContext.cs#L11-L54' title='File snippet `customdbcontext` was extracted from'>snippet source</a> | <a href='#snippet-customdbcontext' title='Navigate to start of snippet `customdbcontext`'>anchor</a></sup>
<!-- endsnippet -->


## Security contact information

To report a security vulnerability, use the [Tidelift security contact](https://tidelift.com/security). Tidelift will coordinate the fix and disclosure.


## Icon

[Database](https://thenounproject.com/term/database/310841/) designed by [Creative Stall](https://thenounproject.com/creativestall/) from [The Noun Project](https://thenounproject.com/creativepriyanka).
