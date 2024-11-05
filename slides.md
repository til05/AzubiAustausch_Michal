---
transition: slide-up
---

# Michal Frydrychowicz
Auszubildender der Anwendungsentwicklung im dritten Ausbildungsjahr<br>

Bei apetito seit<br>
August 2022<br>

An welchen Projekten/Produkten arbeite ich gerade?<br>
Weiterentwicklung einer applikationsübergreifenden Löschfunktion DSGVO-relevanter Daten<br> 

Worüber plaudere ich gerne beim Kaffee?<br>
Volleyball, Ausbildung, IT<br>

---

# Abschlussprojekt

<style>
h1 {
  display: flex;
  justify-content: center; /* Center horizontally */
  align-items: center; /* Center vertically */
  height: 40vh; /* Full height of the viewport */
}
</style>

---
layout: image
image: /Seguenzdiagramm.png
backgroundSize: contain
---

# Sequenzdiagramm – Ablauf einer DSGVO Anfrage

![Sequenzdiagramm](/Sequenzdiagramm.png)


---

# Registrierung einer Applikation 
```csharp
public class RegisterGdprApplicationBackgroundService : BackgroundService
{
  private readonly IMessageSession _messageSession;

  public RegisterGdprApplicationBackgroundService(IMessageSession messageSession)
  {
      _messageSession = messageSession;
  }

  protected override async Task ExecuteAsync(CancellationToken stoppingToken)
  {
      var command = new RegisterGdprApplication
      {
          ApplicationName = "apetito.Customers",
          NServiceBusEndpoint = EndpointName.CustomerService,
          Timestamp = DateTimeOffset.UtcNow
      };

      var options = new SendOptions();
      options.SetDestination(Constants.NServiceBus.EndpointName);
      await _messageSession.Send(command, options);
      await Task.CompletedTask;
  }
}
```

---


# Datenanforderung mehrerer Applikationen 


```csharp
public async Task Handle(DisclosureSagaStarted message, IMessageHandlerContext context)
{
    foreach (var application in Data.Applications)
    {
        if (application.CommandSentSuccessful) continue;
        var options = new SendOptions();
        options.SetDestination(application.NServiceBusEndpoint);

        var applicationDisclosureRequest = new RequestApplicationDisclosure
        {
            CustomerNumber = Data.CustomerNumber
        };

        await context.Send(applicationDisclosureRequest, options);
        application.CommandSentSuccessful = true;
    }
}
```

---

```csharp

public async Task Handle(RequestApplicationDisclosureSuccessful message, IMessageHandlerContext context)
{
      logger.LogInformation(
          "Received RequestApplicationDisclosureSuccessful for customer number {CustomerNumber} from {ApplicationName}",
          message.CustomerNumber, message.ApplicationName);
          
      Data.SuccessfulMessages.Add(mapper.Map<DisclosureSuccessfulMessageDto>(message));
      
      var cachedData = await GetDisclosureDataFromCache();    
      
      cachedData[message.ApplicationName] = message.Data;    
      
      await SetDisclosureDataInCache(cachedData);
}

```

---

# Abruf des Berichts

```csharp
public async Task Handle(GetDisclosureReport message, IMessageHandlerContext context)
{    
    logger.LogInformation("Received GetDisclosureReport for customer number {CustomerNumber}",        
        message.CustomerNumber);    
    var totalApplications = Data.Applications.Count;    
    var successfulApplications = Data.SuccessfulMessages.Select(msg => msg.ApplicationName).Distinct().Count();    
    
    var allApplicationsSuccessful = totalApplications == successfulApplications;    
    var disclosureReportResponse = new DisclosureReportResponse    
    {        
        CustomerNumber = Data.CustomerNumber,        
        IsRequestSuccessful = allApplicationsSuccessful,        
        Data = allApplicationsSuccessful ? await GetDisclosureDataFromCache() : new Dictionary<string, string>()    
    };    
    logger.LogInformation("Sending DisclosureReportResponse for customer number {CustomerNumber}",        
        message.CustomerNumber);    
        
    	await context.Reply(disclosureReportResponse);} 

```
---

# Q&A
Gibt es noch weitere Fragen? Ich beantworte sie gerne!




