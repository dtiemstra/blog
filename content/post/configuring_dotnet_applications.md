---
title: 'Configuring your .Net applications'
date: 2023-04-07
author: 'Diederik Tiemstra'
tags: ['.net', 'configuration']
categories: ['Tech']
draft: false
---

Configuration is a crucial aspect of any software application. It allows developers to separate application logic from its configuration, making it easier to manage, maintain, and deploy the application. In .NET applications, configuration is typically managed using the ConfigurationManager class, which can be used to make settings from different sources available to your application. The order in which these sources are registered is important because your application starts, it loads the configuration providers in the order they are configured. Furthermore, if a configuration source is loaded and the key already exists in a configuration provider that is registered earlier, it overwrites the previous value.

![validation](/img/configuration/providers.png)

### Validating your settings

The .NET Options framework is a powerful and flexible feature that allows developers to configure their .NET applications with ease. It simplifies the process of reading configuration data from various sources such as environment variables, command-line arguments, and JSON files, and makes it available to the application through strongly typed classes.

The Options framework also supports validation of configuration data, which helps to catch errors early in the development process. It allows developers to define constraints on configuration options, such as minimum and maximum values, required fields, and regular expression patterns. If any of the validation rules fail, the framework throws an exception, making it easy to diagnose and fix the issue.

The OptionBuilder class in the .NET Options framework contains several (extension)methods for validating configuration options, such as Validate, ValidateDataAnnotations and ValidateOnStart.

While you can certainly write your own custom validation logic using the Validate and ValidateWith methods, the ValidateDataAnnotations method provides an easy way to leverage the power of the System.ComponentModel.DataAnnotations namespace. This namespace contains a set of validation attributes, such as RequiredAttribute, StringLengthAttribute, and RegularExpressionAttribute, which can be applied to properties in your configuration class to enforce validation rules.

To use DataAnnotations for configuration validation, you need to create a configuration class with properties that represent the configuration settings and decorate the properties with specific attributes. For example, if your application needs some custom settings, you might create a class like this:

```
public class MySettings
{
    [Required]
    public string MySetting1 { get; set; } = string.Empty;

    [Range( 1, 10)]
    public int MySetting2 { get; set; }
}
```

In this example, the MySetting1 property is marked as required using the Required attribute. This means that if this property is not present in one of the configurationproviders, an exception will be thrown when the configuration class is loaded. The MySetting2 property must have a configured value between 1 and 100.

![validation](/img/configuration/data-annotations.png)

By using DataAnnotations for configuration validation, you can ensure that your application's configuration is always valid and complete. This can help prevent errors and improve the overall stability and reliability of your application.

### But be aware......

While DataAnnotations are a useful tool for validating configuration settings in .NET applications, it's important to note that they cannot be used to validate nested objects.

In other words, if your configuration class contains properties that are themselves complex objects, you cannot apply DataAnnotations to those nested objects.

For example, let's change our configuration class a little bit and add a property of the same MySettings type.

```
public class MySettings
{
    [Required]
    public string MySetting1 { get; set; } = string.Empty;

    [Range( 1, 10)]
    public int MySetting2 { get; set; }

    public MySettings NestedSettings { get; set; }
}
```

If we provide the following configuration to the application we expect an exception that the NestedSettings.MySettings1 is required just as we saw in the example earlier.

```
{
  "MySettings": {
    "MySetting1": "MySetting1Value",
    "MySetting2": "MySetting2Value",
    "NestedSettings": {
      "MySetting2": "MySetting2Value"
    }
  }
}
```

However, in this case the application starts without throwing an exception. As mentioned before, this is because DataAnnotion attributes on nested objects are not evaluated.
In this scenario it's up to you to write your own custom validation logic. More information (and a solution to this problem) can be found on the [Microsoft .NET Github page](https://github.com/dotnet/runtime/issues/36093)
