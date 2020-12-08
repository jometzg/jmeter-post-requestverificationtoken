# JMeter how to do POST requests that need a requestverificationtoken
JMeter is a really useful load testing tool. There is an Azure sample https://github.com/Azure-Samples/jmeter-aci-terraform that allows you to automate the process of creating load test infrastructure using Azure DevOps and Azure Container Instances (ACI).

Web applications that implement cross site request forgery countermeasures often create difficulties when recording test scripts as these merely record and playback the same countermeasure tokens, causing the HTTP POST requests to fail.

This repository shows you how you can amend the requests in JMeter to send the correct token on each form post. It implemnents the adivce in this stack overflow article https://stackoverflow.com/questions/26482314/jmeter-token-value-extraction

## Sample Application
The web page
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

The controller
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
