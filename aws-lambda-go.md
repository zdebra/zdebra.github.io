---
title: "AWS Lambda a Golang"
date: 2018-01-28T16:45:55+02:00
draft: false
---

[Lambda](https://aws.amazon.com/lambda/) je služba od AWS, která slouží ke spouštění funkcí v cloudu. Jedná se o tzv. serverless architekturu, kdy k vykonání určitého kódu nepotřebujete složitě vytáčet virtuální mašinu. Namísto toho nadefinujete trigger (spouštěč) a svou funkci. Alternativou od jiného poskytovatele, než je Amazon, jsou [Google Cloud Functions](https://cloud.google.com/functions/docs/) případě [Azure Functions](https://azure.microsoft.com/en-us/services/functions/) od Microsoftu.

Kromě jednoduchosti celé služby je velkou výhodou také cena, kdy platíte za výpočetní čas v kombinaci s využitou pamětí vaší funkce. Amazon dokonce do určitého počtu requestů nechce vůbec nic. Nevýhodou je horší startup time, tedy čas, než AWS alokuje prostor pro vaší funkci. 

# Lambda funkce
Tenhle článek vznikl, protože Amazon nedávno oznámil oficiální poporu jazyka Go pro Lambdu. Ten se tak přidává k již podporovaným 
NodeJS, Pythonu, Javě a C#. Golang je kompilovaný, takže vaší funkci dodáte ve zkompilované binárce. 

Dle [manuálu](https://aws.amazon.com/blogs/compute/announcing-go-support-for-aws-lambda/) stačí importnout [github.com/aws/aws-lambda-go](https://github.com/aws/aws-lambda-go) a v main fuknci spustit `lambda.Start(handler)`, kde handler je libovolná funkce, pro kterou platí:
1. žádný nebo 1 vstupní parametr (případně 1 vstupní parametr a `context.Context`)
2. žádný výstup nebo `error` (případně `error` a jeden výstupní parametr)

# Využítí Lambdy
Jako příklad využítí ukážu využití lambdy jako intergraci Healhchecku se Slackem. Healthcheck je služba od AWS, která sleduje, jestli jsou vaše servery naživu. Můžete si nadefinovat různé alarmy - například aby se alarm rozezvučel při vysoké latenci při komunikaci s vaším endpointem, nebo při určitých HTTP statusech. Konfigurací je mnoho. 

## Klikání v konzoli
V AWS konzoli pod [Route 53](https://aws.amazon.com/route53/) vyrobte nový Healthcheck a v první části wizzardu mu nadefinujte kýžené atributy. V dalším kroku nadefinujete kam se případný alarm bude hlásit. K tomu využijeme další AWS službu, [Simple Notifications Service](https://aws.amazon.com/sns/). SNS je služba, využívající publish/subsribe pattern. V našem případě je v SNS vytvořený určitý topic, do kterého Healhcheck zapíše, pokud se splní nějaká podmínka nadefinovaného Alarmu. To právě uděláme v druhém kroku onoho wizzardu, takže do SNS Dashboardu ani nemusíme chodit.

Zbytek naklikáme v dashboardu Lambdy. Vytvoříme novou funkci a jako trigger zvolíme SNS a v konfiguraci navolíme topic, který vygeneroval Healthcheck wizzard.

## Implementace funkce
```go
package main

import (
	"errors"
	"log"

	"bytes"
	"encoding/json"
	"fmt"
	"github.com/aws/aws-lambda-go/lambda"
	"net/http"
	"os"
)

const slackWebhookKey = "WEBHOOK_URL"

var (
	ErrInvalidStatusCode    = errors.New("invalid status code")
	ErrSlackWebhookNotFound = errors.New("slack webhook not found in env variables")
)

func Handler(request SNSMessage) error {
	log.Printf("processing message from SNS: %v\n", request)
	if err := request.Validate(); err != nil {
		return err
	}
	slackURL, found := os.LookupEnv(slackWebhookKey)
	if !found {
		return ErrSlackWebhookNotFound
	}
	input := request.Records[0].SNS
	payload := SlackPayload{
		Text: fmt.Sprintf("%s: %s", input.Subject, input.Message),
	}
	payloadJSON, _ := json.Marshal(payload)
	resp, err := http.Post(slackURL, "application/json", bytes.NewBuffer(payloadJSON))
	if err != nil {
		return err
	}
	if resp.StatusCode != http.StatusOK {
		return ErrInvalidStatusCode
	}
	log.Printf("message with id %s send", input.MessageID)
	return nil
}

func main() {
	lambda.Start(Handler)
}
```

Celá implementace: [https://github.com/zdebra/lambdaSNSToSlack](https://github.com/zdebra/lambdaSNSToSlack).

## Slack Webhook URL

Tu získáte velmi jednoduše v dashboardu vaší Slack organizace. Vytvoříte nový Incoming webhook a dostanete tuto URL. Funkce ji očekává jako env proměnou `WEBHOOK_URL` (zadáte v konfiguraci lambdy).

Doporučuji si to forknout a upravit text chodící do Slacku, protože tato raw message, kterou tam posílám já, není moc čitelná.

# Závěry
Lambda je super jednoduchý, levný a bezúdržbový způsob jak vykonat nějaký kód. Zkompilovaná binárka golangu je navíc o dost rychlejší, než spouštění funkcí v jiných jazycích.