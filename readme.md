>Etapes

1.Installer DnSpy

2.Fermer les services de netlimiter en cours (avec cmd en tant qu'admin): 
```cmd
taskkill /f /im NLClientApp.exe
net stop nlsvc
```
3.Ouvrir DnSpy en tant qu'administrateur

4.Ouvrez le fichier NetLimiter.dll de l'utilisateur (en tant qu'utilisateur standard situé sous " C:\Program Files\Locktime Software\NetLimiter\NetLimiter.dll"). Il est possible d'ouvrir votre répertoire actuel afin de trouver des programmes de bibliothèque. Pour cela, choisissez "Fichier" => Ouvrir.

5.Après l'ouverture , ouvrez NetLimiter (version) => NetLimiter.dll => {} NetLimiter.Service => NLLicense

6.Ensuite, vous pouvez trouver la classe NLLicense dans votre ancien constructeur NLLicense().

//Original
```C#
public NLLicense()
{
	this.Expiration = DateTime.MaxValue;
	this.EditionId = "pro";
	this.Quantity = 1;
	this.IsRegistered = false;
	this.IsRecurring = false;
	this.SupporetedFeatures = new SupportedFeatures(this.EditionId);
	this.InitTestingVersion();
}
```

Il n'est pas possible de définir de manière permanente this.IsRegistered sur false sur true. Pour cela, appuyez sur le bouton "Modifier cette méthode C#". Après l'installation, vous pouvez appuyer sur le bouton "Compile"


//Edited
```C#
public NLLicense()
{
	this.Expiration = DateTime.MaxValue;
	this.EditionId = "pro";
	this.Quantity = 1;
	this.IsRegistered = true;
	this.IsRecurring = false;
	this.SupporetedFeatures = new SupportedFeatures(this.EditionId);
	this.InitTestingVersion();
}


```
Après avoir configuré NLLicense(), vous serez dans la section « Démarreur d'entreprise » et ouvrez NetLimiter (version) => NetLimiter.dll => {} NetLimiter.Service => NLServiceTemp
La classe NLServiceTemp est la fonction InitLicense()
//Original

```C#
private void InitLicense()
{
	string licensePath = this.GetLicensePath();
	NLServiceTemp._logger.LogInformation("RegData path: {path}", new object[] { licensePath });
	RegData regData = null;
	if (File.Exists(licensePath))
	{
		string text = File.ReadAllText(licensePath);
		try
		{
			regData = JsonConvert.DeserializeObject<RegData>(text);
			if (!this.VerifyRegData(regData))
			{
				regData = null;
			}
		}
		catch (Exception ex)
		{
			NLServiceTemp._logger.LogError(ex, "Failed to init existing license: {path}", new object[] { licensePath });
		}
	}
	if (regData != null)
	{
		this.License = new NLLicense(regData);
		NLServiceTemp._logger.LogInformation("License found: expiration={expiration}", new object[] { this.License.Expiration });
	}
	else
	{
		DateTime installTime = this.GetInstallTime();
		DateTime dateTime = installTime.AddDays(28.0);
		this.License = new NLLicense(dateTime);
		NLServiceTemp._logger.LogInformation("License not found: expiration={expiration}, installTime={installTime}", new object[]
		{
			this.License.Expiration,
			installTime
		});
	}
	this.Callback.OnLicenseChange(this.License);
}
```

>Utilisez la commande "DateTime dateTime = installTime.AddDays(28.0);" et indique "28.0" sur "99999.0". Cela est nécessaire pour l'évaluation du produit. Il est donc nécessaire de choisir ce que vous devez utiliser System.Exception en utilisant l'option "catch (Exception ex)" pour définir l'option lors de la compilation des éléments. é.
```C#
//Edited
private void InitLicense()
{
	string licensePath = this.GetLicensePath();
	NLServiceTemp._logger.LogInformation("RegData path: {path}", new object[] { licensePath });
	RegData regData = null;
	if (File.Exists(licensePath))
	{
		string text = File.ReadAllText(licensePath);
		try
		{
			regData = JsonConvert.DeserializeObject<RegData>(text);
			if (!this.VerifyRegData(regData))
			{
				regData = null;
			}
		}
		catch (System.Exception ex)
		{
			NLServiceTemp._logger.LogError(ex, "Failed to init existing license: {path}", new object[] { licensePath });
		}
	}
	if (regData != null)
	{
		this.License = new NLLicense(regData);
		NLServiceTemp._logger.LogInformation("License found: expiration={expiration}", new object[] { this.License.Expiration });
	}
	else
	{
		DateTime installTime = this.GetInstallTime();
		DateTime dateTime = installTime.AddDays(99999.0);
		this.License = new NLLicense(dateTime);
		NLServiceTemp._logger.LogInformation("License not found: expiration={expiration}, installTime={installTime}", new object[]
		{
			this.License.Expiration,
			installTime
		});
	}
	this.Callback.OnLicenseChange(this.License);
}
```