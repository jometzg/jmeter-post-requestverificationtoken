# JMeter - POST requests with __RequestVerificationToken
JMeter is a really useful load testing tool. There is an Azure sample https://github.com/Azure-Samples/jmeter-aci-terraform that allows you to automate the process of creating load test infrastructure using Azure DevOps and Azure Container Instances (ACI).

Web applications that implement cross site request forgery countermeasures often create difficulties when recording test scripts as these merely record and playback the same countermeasure tokens, causing the HTTP POST requests to fail.

This repository shows you how you can amend the requests in JMeter to send the correct token on each form post. It implemnents the adivce in this stack overflow article https://stackoverflow.com/questions/26482314/jmeter-token-value-extraction

## Sample Application
The web page:
```
<form method="post" enctype="multipart/form-data" asp-controller="Form" asp-action="Index">
    <table>
        <tr>
            <td>First Name: </td>
            <td><input type="text" id="txtFirstName" name="FirstName" /></td>
        </tr>
        <tr>
            <td>Last Name: </td>
            <td><input type="text" id="txtLastName" name="LastName" /></td>
        </tr>
        <tr>
            <td></td>
            <td><input type="submit" value="Submit" /></td>
        </tr>
    </table>
    <hr />
</form>
```
When this web page gets generated on the web server, an extra field "__RequestVerificationToken" is added to the HTML form which has been generated using a key that resides in the web app. A form POST using this token is required to provide that the POST was from the same site.

The controller is decorated with *ValidateAntiForgeryToken* which tells the controller to evaulate whether the value of the extra field is correct and if not throws an exception.
```
[HttpPost]
[ValidateAntiForgeryToken]
public ActionResult Index(IFormCollection collection)
{
   try
      {
         ViewBag.Name = string.Format("Name: {0} {1}", collection["FirstName"], collection["LastName"]);
         return View();
      }
      catch
      {
         return View();
      }
}
```
## Load Test Problems
Load tests generally record a series of HTTP requests and play them back. The __RequestVerificationToken will be sent on any later POST requests, but it will fail the validation and so will be rejected by the web server (quite rightly).

![Rejected request](/rejected-request.png)

So what we need to do is to extract the token from the preceding page and then apply that token to the subsequent POST request.

## The solution
The solution is to add a POST request *CSS Selector Extractor* to the previous page (the one that generated the HTML form) and then put this value in a variable to then be later used in the next POST request:

![CSS Selector Extractor](/css-selector-extractor.png)
As can be seen above, the selector finds the input field __RequestVerficationToken and then puts its *value* into a variable, which we have named *token*

![Updated post request](/response2.png)
The value of token using the expression ${__V(token)} is applied to the POST request.

The result is a "green" response.

![Response from post request](/green-response.png)

## Try it yourself
This repository has a sample JMeter test plan (JMX) [PostFormTest.jmx](postformtest.jmx) that illustrates how to implement this. In addition is the sample application code that can be used to validate this test script.
