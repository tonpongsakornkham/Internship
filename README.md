# งานนี้ทำตอนฝึกงานไม่สามารถเปิดเผยตัวโปรเเกรมได้จึงนำเอกสาร ReadMe ที่ข้าพเจ้าได้เขียนการทำงานไว้มาโชว์
# App Store Connect API
Automate tasks on the Apple Developer website and App Store Connect.I create link in **navBar**

**Requirement**
1.  [ BouncyCastle](https://www.nuget.org/packages/BouncyCastle/1.8.5?_src=template) use to generate securitykey
2.  [JWT](https://www.nuget.org/packages/JWT/5.2.2?_src=template) for create and authorize Token before to use **App Store Connect API**.
3.  [Microsoft.AspNetCore.Authentication.JwtBearer](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.JwtBearer/2.2.0?_src=template) Authorize token and show you Token create.
4.  [RestSharp](https://www.nuget.org/packages/RestSharp/106.6.10?_src=template) use to add header for authorize in your **Project** 

**Step of Work**
1. Creating **API Keys** for App Store Connect API read here [Create API keys used to sign JWTs and authorize API requests](https://developer.apple.com/documentation/appstoreconnectapi/creating_api_keys_for_app_store_connect_api)
2.  **Generating Tokens** for API Requests .I will create JSON Web Tokens signed with private key to authorize API requests read Example Here [ Create the JWT & Sign JWT](https://developer.apple.com/documentation/appstoreconnectapi/generating_tokens_for_api_requests)
3.  Test **GET**  ```https://api.appstoreconnect.apple.com/v1/apps``` with ```Authorization: Bearer [signed token]``` in **Postman** or etc..


 **SOL Step**
 1.  Create ```Controller``` for **GetToken** and show data in **Web_Api** 
    2. Get your private key and Convert to  **CngKey**  
   ```
    var Conkey = CngKey.Import(Convert.FromBase64String(privateKey),
                        CngKeyBlobFormat.Pkcs8PrivateBlob);
   ```

3. Convert your **CngKey** to **ecdsaCng** ,Before Signing Key , put **PublicKey** 
and generate your **SigningCredentials**

```
var ecdsaCng = new ECDsaCng(Conkey);
var securityKey = new ECDsaSecurityKey(ecdsaCng) { KeyId = "PublicKey" };
var credentials = new SigningCredentials(securityKey, "ES256"); 
  ``````
  *NOTE:* **All JWTs for App Store Connect API must be signed with ES256 encryption**
  
  4. Now you can Create Token with
	  ``````
	   var token = new JwtSecurityToken(
                issuer: Issuer,
                audience: Audience,
                expires: DateTime.Now.AddMinutes(20),
                signingCredentials: credentials
            );
		return new JwtSecurityTokenHandler().WriteToken(token);
		``````
  *NOTE:* **The token's expiration time, in Unix epoch time; tokens that expire more than 20 minutes in the future are not valid**

5. Create Authorize with use **RestSharp** 
	 ``````
	 var client = new RestClient("https://api.appstoreconnect.apple.com/v1/apps");
     var request = new RestRequest(Method.GET);
	     request.AddHeader("cache-control", "no-cache");
         request.AddHeader("Authorization", "Bearer " + GetToken());
     IRestResponse response = client.Execute(request);
     
     return response.Content;
      ``````

*NOTE*: **Get Finance report change client to** 
```
https://api.appstoreconnect.apple.com/v1/financeReports
```
  
# Test Receipt Validate (Mock)
  Validating Receipts With the App Store 
  

*NOTE*: **The receipt validate only ,It don't have get receipt from appstore.**
Example with ***Receipt data :***
```
receiptData = "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";
```
1. Submit this **JSON object** as the payload of an **HTTP POST** request. In the test environment, use  `https://sandbox.itunes.apple.com/verifyReceipt`  as the URL. In production, use  `https://buy.itunes.apple.com/verifyReceipt`  as the URL.
2. Example code to convert  **JSON object**
	```
	var json = new JObject(new JProperty("receipt-data", receiptData)).ToString();
	```
3. You can **POST RESPONCE** same **Authorize**  *See SOL Step 5 in App Store Connect API*
	Example Other Code :
	```
	var request = System.Net.HttpWebRequest.Create("https://sandbox.itunes.apple.com/verifyReceipt");
                request.Method = "POST";
                request.ContentType = "application/json";
                request.ContentLength = postBytes.Length;
	```
4. Use **Stream** to get **returnMassage** from **receipt data**
	```
	using (var stream = request.GetRequestStream())
                {
                    stream.Write(postBytes, 0, postBytes.Length);
                    stream.Flush();
                }

                var sendresponse = request.GetResponse();

                string sendresponsetext = "";
    using (var streamReader = new StreamReader(sendresponse.GetResponseStream()))
                {
                    sendresponsetext = streamReader.ReadToEnd().Trim();
                }
                returnmessage = sendresponsetext;
	```
	
# Download Sales and Trends reports(CMD)
Detail report you can check [Here](https://help.apple.com/itc/appsreporterguide/#/itcbe21ac7db)

**Solution :**
1. Download Reporter [THIS](https://itunespartner.apple.com/assets/downloads/Reporter.zip). You have recieve package contains two files -   
	- `Reporter.jar`
	-   `Reporter.properties`

2. Sign in to Reporter , Open the Reporter.properties file using a text editor and Write your access token. 
3. You can read GENERAL COMMANDS in [Here](https://help.apple.com/itc/appsreporterguide/#/itc469b4b7eb)
4. Download Sales and Trends reports  **Syntax**

	`$ java -jar Reporter.jar p=[properties file] Sales.getReport [vendor number], [report type], [report subtype], [date type], [date], [version]* (if applicable)`

	Example:  
	`$ java -jar Reporter.jar p=Reporter.properties Sales.getReport 80012345, Sales, Summary, Daily, 20150201`


# Download Sales and Trends reports(C#)

I create WebForm and download with CMD on WebFrom

**Code to execute cmd on C#:**
```
Process p = new Process();
p.StartInfo.WorkingDirectory = @"C:\Users\Ton\Desktop\Reporter";
p.StartInfo.UseShellExecute = false;
p.StartInfo.FileName = "java";
p.StartInfo.Arguments = @"-jar Reporter.jar p=Reporter.properties Sales.getReport 85588896, Sales, Summary, Daily, 20190715";
p.StartInfo.RedirectStandardInput = true;
p.StartInfo.RedirectStandardOutput = true;
p.StartInfo.RedirectStandardError = true;
p.Start();
p.WaitForExit();
 ```
 *NOTE:* Set you **WorkingDirectory** to your place **Reporter File** . After run it, File will **download** in to your **WorkingDirector**y of your path **Reporter File**.
 Download Sales and Trends reports  **Syntax**

	`$ java -jar Reporter.jar p=[properties file] Sales.getReport [vendor number], [report type], [report subtype], [date type], [date], [version]* (if applicable)`

# Extract .gz File c#

When download Sale Report .gz I will add code to extract it .

**[First]** : Search Path file form Directory Path

**Code:** 
```
DirectoryInfo directorySelected = new DirectoryInfo(directoryPath);
foreach (FileInfo fileToDecompress in directorySelected.GetFiles("*.gz"))
     {
        Decompress(fileToDecompress);
     }
```
***NOTE:*** Get file all .gz in **Directory Path**

**[Second]** : Decompress .gz file 

**Code:** 
```
public static void Decompress(FileInfo fileToDecompress)
   {
     using (FileStream originalFileStream = fileToDecompress.OpenRead())
	   {
          string currentFileName = fileToDecompress.FullName;
          string newFileName = currentFileName.Remove(currentFileName.Length - fileToDecompress.Extension.Length);
          using (FileStream decompressedFileStream = File.Create(newFileName))
             {
               using (GZipStream decompressionStream = new GZipStream(originalFileStream, CompressionMode.Decompress))
                  {
                     decompressionStream.CopyTo(decompressedFileStream);
                     Console.WriteLine($"Decompressed: {fileToDecompress.Name}");
                  }
             }
        }
     }
```
***NOTE:*** File will extract in **Directory Path**

**[Read Text File(MOCK)]** : ```string text = File.ReadAllText(pathToFile, Encoding.UTF8);``` 


# Ton_wealth How to add in Android Studio

### This project created when Internship in Wealth

**How To add Git into AndroidStudio**
1. OpenProject File > Setting > VersionControl
2. Choose GitHub and add ID/PassWord of GitHub and testConnection 
3. OpenProject VCS > Import into Version Control > Share Project on GitHub 

**What you need to know run this Project**
1. Open an existing Android Studio 
2. Select from your path file
3. **Minimum Required Android 6.0.1 ,API 23 to Run this Program** 
4. Add maven in build.gradle(Project)
```java
repositories {
        google()
        jcenter()
        maven { url 'https://jitpack.io' }

    }
 ```
 5. Lib to use this Project
 ```java
    implementation 'com.github.PhilJay:MPAndroidChart:v3.1.0'
    implementation 'com.github.vihtarb:tooltip:0.2.0'
    implementation 'me.toptas.fancyshowcase:fancyshowcaseview:1.1.5'
    implementation "org.jetbrains.kotlin:kotlin-stdlib:1.3.31"
    implementation 'com.getkeepsafe.taptargetview:taptargetview:1.12.0'
 ```
6. Build Gradle to use this Project
```java
dependencies {
        classpath 'com.android.tools.build:gradle:3.4.1'
    }
```

**7. Link All Library in use**

1. MPAndroidChart (**graphPage.java**)
```
implementation 'com.github.PhilJay:MPAndroidChart:v3.1.0'

```
>  **[https://github.com/PhilJay/MPAndroidChart](https://github.com/PhilJay/MPAndroidChart)**

2. Android Tooltips (**FancyShowcase.java**, **customDialog.java**)
```
implementation 'com.github.vihtarb:tooltip:0.2.0
```
> **[https://github.com/ViHtarb/Tooltip](https://github.com/ViHtarb/Tooltip)**

3. FancyShowCaseView (**FancyShowcase.java**)
```
implementation 'me.toptas.fancyshowcase:fancyshowcaseview:1.1.5' 
```
> **[https://github.com/faruktoptas/FancyShowCaseView](https://github.com/faruktoptas/FancyShowCaseView)**

4. TapTargetView (**customDialog.java**)
```
 implementation 'com.getkeepsafe.taptargetview:taptargetview:1.12.0'
```
> **[https://github.com/KeepSafe/TapTargetView](https://github.com/KeepSafe/TapTargetView)**

5. Kotlin Interface
```
implementation "org.jetbrains.kotlin:kotlin-stdlib:1.3.31"
```

## Android BlockChain with Ethereum

**What I Do ?**
1.  Generate a contracts **ABI** and **binary** files from a **Solidity File**.
2.  Use  **web3j**  libraries to generate java wrapper classes from a smart contract’s methods.
3.  Use  **Infura**  to host an **Ethereum** node remotely that allows us to complete transactions and interact with the blockchain.
4.  Verify a transaction was made via the **Etherscan blockchain** explorer.


**First Step to Run**
1. add Libraries  [Web3j](https://github.com/web3j/web3j) in your project 
2.  Create Account in [**Infura**](https://infura.io/) and Create new project on **Infura**
3. Install  [Solidity](https://solidity.readthedocs.io/en/v0.5.10/index.html) Check Version  ```solc --version. ``` in command line .If it’s not installed, install it globally. ```npm install -g solc```
4. Load **web3j** and paste inside your project ,Check ```web3j --version```

 **Create a Wallet with Metamask on the Rinkeby Testnet**
 1.  First we will need to download the official Metamask browser extension.  
    [https://metamask.io/](https://metamask.io/). and create your wallet
    2. Click to Icon **Metamask** change from **Main Ethereum Network** to **Rinkeby Test Network**
  
  **Request Ether from the Rinkeby Faucet**
  1. On a social media profile of your choice, make a post with your public address.  **MAKE SURE THE POST CONTAINS YOUR PUBLIC ADDRESS, NOT YOUR PRIVATE KEY.**
  2. Copy and paste the URL to this post inside of the  _Give me Ether_  Field.
  
  
  **Deploy the Contract**
  1. Head to [http://remix.ethereum.org](http://remix.ethereum.org/) and **paste with your solidity contract** in here
  2. When Success compiler go to verify in **Etherscan** with your address
  
  **Generate File with Solidity**
  1.  Generate your bin and ABI by first copying the solidity file somewhere into your project directory. Type in the following command into your command line. 
  ```
  solc <contract>.sol — bin — abi — optimize -o <output-dir>/
  ```
  2. generate your Java code using 
  ```comand line
  web3j solidity generate -b /path/to/<smart-contract>.bin -a /path/to/<smart-
  contract>.abi -o /path/to/src/main/java -p com.your.organisation.name
  ```

  # Create .obb File and Unzip (In lib2)

**APK Expansion Files**. Google Play requires that the compressed APK that users download be no more than 100MB .So I createed Expansion Files for upload in GoogleStore(APK + File.obb) 

**How toCreate File.obb**
1.  First,I use **WinRar** to archive in Expansion Files and convert it to **File.obb**.Setting Archive to **Storage** only. **0% compression**
2. Unzip File using  **Zip4j** library 
	```
	new ZipFile("filename.zip").extractAll("/destination_directory");
	```
*NOTE:* **Google Play Store when download expansion Files on your devie, It's not to unzip but read and use them.**

You can read about APK Expansion Files is [HERE](https://developer.android.com/google/play/expansion-files.html#Overview)



