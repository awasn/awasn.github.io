---
title: SmtpClient i <mailSettings>
date: 2008-09-18T15:47:00+02:00
lastmod: 2018-09-17T13:21:00+02:00
languageCode: pl-pl
description: Ustawienia klienta pocztowego .NET SmtpClient w pliku konfiguracyjnym aplikacji
categories:
  - Development
tags:
  - .NET
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2008/09/18/smtpclient-i-mailsettings.aspx
disqus_disable: true
---

Czasami małe rzeczy sprawiają dużo radości. Jedną z nich jest możliwość ustawienia klienta pocztowego `SmptClient` poprzez wpisy w pliku konfiguracyjnym aplikacji. Rozważmy przykład:

```csharp
            string mailAddresses = ConfigurationManager.
                AppSettings["mailAddresses"];
            if (string.IsNullOrEmpty(mailAddresses)){
                return;
            }

            MailMessage message = new MailMessage();
            message.From = new MailAddress("wnioski@homski.pl", "Wnioski");
            foreach (string mailAddress in mailAddresses.Split(';')){
                message.To.Add(new MailAddress(mailAddress));
            }
            message.SubjectEncoding = Encoding.UTF8;
            message.Subject = "Tytuł wiadomości";
            message.BodyEncoding = Encoding.UTF8;
            message.Body = "Treść wiadomości";

            SmtpClient client = new SmtpClient();
            client.Send(message);
```

Z pliku **config** aplikacji pobieram informację o adresach, do których należy pocztę wysłać. Następnie buduję wiadomość i wysyłam. Nie konfiguruję w kodzie programu zupełnie danych serwera SMTP, który posłuży do wysłania poczty. Zamiast tego pozwalam klasie `SmptClient` na samodzielne pobranie tych informacji z konfiguracji:

```xml
          <?xml version="1.0" encoding="utf-8" ?>
          <configuration>
            <system.net>
              <mailSettings>
                <smtp deliveryMethod="Network">
                  <network host="smtp.homski.pl"
                          userName=""
                          password=""/>
                </smtp>
              </mailSettings>
            </system.net>

            <appSettings>
              <add key="mailAddresses"
                  value="arkadiusz.wasniewski@email.pl"/>
            </appSettings>
          </configuration>
```

Szczegóły schematu `network` (oraz elementów nadrzędnych) można znaleźć na stronach [msdn](https://docs.microsoft.com/en-us/dotnet/framework/configure-apps/file-schema/network/network-element-network-settings).