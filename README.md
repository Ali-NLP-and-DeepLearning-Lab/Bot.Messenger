# Bot.Messenger
.NET SDK Library for Facebook's Messenger Bot

Sample WebAPI project https://github.com/olisamaduka/MessengerBot-WebAPI

## How to Use

- Create an instance of the Bot.Messenger.MessengerPlatform class and pass it your credentials

```csharp
        MessengerPlatform bot = MessengerPlatform.CreateInstance(
                MessengerPlatform.CreateCredentials(_appSecret, _pageToken, _verifyToken));
```

- OR set your credentials in web.config and initialize in code as so;

    > Web.config

    ```markup
    <configuration>
      <configSections>
        <sectionGroup name="applicationSettings" type="System.Configuration.ApplicationSettingsGroup, System, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089">
          <section name="Bot.Messenger.Settings" type="System.Configuration.ClientSettingsSection, System, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />
        </sectionGroup>
      </configSections>
      <applicationSettings>
        <Bot.Messenger.Settings>
          <setting name="PageToken" serializeAs="String">
            <value>page token</value>
          </setting>
          <setting name="AppSecret" serializeAs="String">
            <value>app secret</value>
          </setting>
          <setting name="VerifyToken" serializeAs="String">
            <value>hello</value>
          </setting>
        </Bot.Messenger.Settings>
      </applicationSettings>
      ...
    </configuration>
    ```

  > Code

    ```csharp
            MessengerPlatform bot = MessengerPlatform.CreateInstance();
            // OR
            // MessengerPlatform bot = new MessengerPlatform();
    ```

> Credentials are fetched from web.config ApplicationSettings when the CreateInstance method is called without a credentials parameter or if the parameterless constructor is used to initialize the MessengerPlatform class. This holds true for all types that inherit from Bot.Messenger.ApiBase.
>

- You can access the Messenger platform APIs through an instance of the Bot.Messenger.MessengerPlatform class (APIs supported are Send API, User Profile API and Messenger Profile API).  Here is a sample usage at an ASP.NET WebAPI WebhookController 

```csharp
        [HttpPost]
        public async Task<HttpResponseMessage> Post()
        {
            var body = await Request.Content.ReadAsStringAsync();

            if (!_Bot.Authenticator.VerifySignature(Request.Headers.GetValues("X-Hub-Signature").FirstOrDefault(), body))
                return new HttpResponseMessage(HttpStatusCode.BadRequest);
            MessengerPlatform bot = MessengerPlatform.CreateInstance(
                    MessengerPlatform.CreateCredentials(_appSecret, _pageToken, _verifyToken));

            Bot.Messenger.Models.WebhookModel webhookModel = bot.ProcessWebhookRequest(body);
            
            foreach (var entry in webhookModel.Entries)
            {
                foreach (var evt in entry.Events)
                {                
                    if (evt.EventType == WebhookEventType.MessageReceivedCallback)
                    {
                        await bot.SendApi.SendActionAsync(evt.Sender.ID, SenderAction.typing_on);

                        Bot.Messenger.Models.UserProfileResponse userProfileRsp = await bot.UserProfileApi.GetUserProfileAsync(evt.Sender.ID);

                        await bot.SendApi.SendTextAsync(evt.Sender.ID, $"Hello {userProfileRsp?.FirstName} :)");
                    }
                }
            }

            return new HttpResponseMessage(HttpStatusCode.OK);
        }
```




