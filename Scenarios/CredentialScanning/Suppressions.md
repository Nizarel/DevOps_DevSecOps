# Supress CredScan errors

Once you run the scanner pipeline the first time, you will need to analyze any failures.  Below is an example using eshopweb where four failures were found.

```Text
Analyzing: D:\a\1\_sdt\logs\CredentialScanner\CredentialScanner-matches.xml


src\Infrastructure\Identity\AppIdentityDbContextSeed.cs(11,): error : SecretinFile :
{{Searcher}}DefaultPassword
{{Code}}See AppIdentityDbContextSeed.cs line 11 for the code resulting in match
{{Info}}Found known password in source file.
{{Suggest}}Validate file contains secrets, remove, roll credential, and use approved store. For additional information on secret remediation see https://aka.ms/credscan



tests\FunctionalTests\Web\Controllers\AccountControllerSignIn.cs(79,): error : SecretinFile :
{{Searcher}}DefaultPassword
{{Code}}See AccountControllerSignIn.cs line 79 for the code resulting in match
{{Info}}Found known password in source file.
{{Suggest}}Validate file contains secrets, remove, roll credential, and use approved store. For additional information on secret remediation see https://aka.ms/credscan



src\Web\Areas\Identity\Pages\Account\Login.cshtml(53,): error : SecretinFile :
{{Searcher}}DefaultPassword
{{Code}}See Login.cshtml line 53 for the code resulting in match
{{Info}}Found known password in source file.
{{Suggest}}Validate file contains secrets, remove, roll credential, and use approved store. For additional information on secret remediation see https://aka.ms/credscan



tests\IntegrationTests\LoginService.cs(42,): error : SecretinFile :
{{Searcher}}DefaultPassword
{{Code}}See LoginService.cs line 42 for the code resulting in match
{{Info}}Found known password in source file.
{{Suggest}}Validate file contains secrets, remove, roll credential, and use approved store. For additional information on secret remediation see https://aka.ms/credscan



CredScan failed with 4 issues found.

4 total issues were found across 1 tools. Failed.
```

To correct the failures, follow guidance ([link to 1es wiki](https://microsoft.sharepoint.com/teams/CESecEngineering/CredScan/CredScan%20Wiki/Suppression%20Usage%20Examples.aspx)) on creating suppressions below:

```Text
[SuppressMessage("Microsoft.Security", "CS002:SecretInNextLine", Justification="….")]

[SuppressMessage\("Microsoft.Security", "CS001:SecretInline", Justification="..."\)]
```

For XML, Html files:

```HTML
<add key="StoreAccountKey" value="$AKV$" /><!--[SuppressMessage("Microsoft.Security", "CS001:SecretInline", Justification="...")]-->

<!--[SuppressMessage("Microsoft.Security", "CS002:SecretInNextLine", Justification="...")]-->
<add key="KeyVaultUri " value="ARISKeyVault.vault.azure.net" />
```

For C#, C++, Java source code files:

```C#
new StorageCredentials("acc", "@KV"); // [SuppressMessage("Microsoft.Security", "CS001:SecretInline", Justification="...")]

// [SuppressMessage("Microsoft.Security", "CS002:SecretInNextLine", Justification="...")]
new StorageCredentials("acc", "@/KeyVault/Value");
```

For Javascript, Typescript, JSON file:
(There is no official support for comments in JSON, please make sure your parser can handle this syntax properly at runtime, otherwise please consider to use Local Suppression feature instead.)

```JSON
"password": "[parameters('pwd)]", // [SuppressMessage("Microsoft.Security", "CS001:SecretInline", Justification="...")]

//[SuppressMessage("Microsoft.Security", "CS002:SecretInNextLine", Justification="...")]
"password": "[parameters('serviceCertificatePassword')]",​
```

​For PowerShell v2+ files:

```POWERSHELL
$conn="Initial Catalog=test;uid=sa;pwd=@Pwd;" <#[SuppressMessage("Microsoft.Security", "CS001:SecretInline", Justification="...")]#>

<#[SuppressMessage("Microsoft.Security", "CS002:SecretInNextLine", Justification="...")]#>
$ConnectionString = "Data Source=test;Initial Catalog=test;User Id=test;Password=@Password;"
```

For PowerShell v1, Python files:

```POWERSHELL
"Initial Catalog=test;uid=sa;pwd=@Pwd;" #[SuppressMessage("Microsoft.Security", "CS001:SecretInline", Justification="...")]

#[SuppressMessage("Microsoft.Security", "CS002:SecretInNextLine", Justification="...")]
"Data Source=test;Initial Catalog=test;User Id=test;Password=@Password;"
```

For T-SQL files:

```SQL
"Initial Catalog=test;uid=sa;pwd=@Pwd;" -- [SuppressMessage("Microsoft.Security", "CS001:SecretInline", Justification="...")]

-- [SuppressMessage("Microsoft.Security", "CS002:SecretInNextLine", Justification="...")]
"Data Source=test;Initial Catalog=test;User Id=test;Password=@Password;" ​
```

For PEM files

```Text
[SuppressMessage("Microsoft.Security", "CS002:SecretInNextLine", Justification="...")]

----- Begin RSA Private Key -----
```

I fixed the above failures using inline or linenext guidance and then reran the pipeline.  All failures were resolved.  Be sure to include justification in the suppression so others know why you suppressed the result.
