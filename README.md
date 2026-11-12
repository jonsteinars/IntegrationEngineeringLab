# IntegrationEngineeringLab
.NET lausn sem tekur við fjárhagsfærslum og kemur þeim á öruggan hátt inn í bókhaldskerfi.

Hvenær er hver lausn best? (API-samþætting vs API-samþætting + gagnagrunnur vs API-samþætting + gagnagrunnur + biðröð).
A. Bein API-samþætting - Einföldust
Ytra API -> Þjónusta -> Bókhald

Hentar best þegar: Gagnamagn er lítið, kerfin eru áreiðanleg, auðvelt er að sækja færslur aftur og viðtakandinn getur tryggt að færslur séu ekki tvíbókaðar.
Kostir: Fáir íhlutir, lítill kostnaður og einfalt viðhald.
Gallar: Án varanlegrar stöðuskráningar getur verið erfitt að vita hvaða færslur hafa farið í gegn og hvaða færslur þarf að reyna aftur.

B. API + gagnagrunnur
Ytra API -> Þjónusta -> Bókhald
          PostgreSQL
    Færslur og vinnslustaða

Hentar best þegar: Við þurfum rekjanleika, endurvinnslu og áreiðanlega skráningu á því hvaða færslur hafa verið mótteknar og afgreiddar.
Kostir: Gagnagrunnurinn getur geymt færslur og vinnslustöðu, og bakgrunnsvinnsla getur fundið óafgreiddar færslur eftir bilun.
Gallar: Við þurfum að hanna stöðubreytingar, gagnagrunnstengingar og endurvinnslu rétt.

C. API + gagnagrunnur + biðröð (Meiri sveigjanleiki)
API -> Biðröð -> Worker
Gagnagrunnur geymir færslur og vinnslustöðu

Hentar best þegar: Miklar sveiflur eru í álagi, margar þjónustur vinna úr sömu upplýsingum, við þurfum sjálfstæða skölun eða viljum aftengja móttöku og vinnslu.
Kostir: Vinnsluþjónustan getur unnið á sínum eigin hraða og kerfið þolir betur tímabundnar álagssveiflur.
Gallar: Fleiri íhlutir, aukin rekstrarkomplexitet og ný vandamál varðandi samræmi gagnagrunns og biðraðar.

2.a) Samanburður á kostum og göllum við að nota sameiginlegt forrtið fyrir API og Worker eða sjálfstætt fyrir hvort:
Atriði	                    A. Eitt forrit	                  B. Tvö forrit
Uppsetning	                   Einfaldari	                       Flóknari
CI/CD	                         Eitt deployment-ferli	           Tvö sjálfstæð ferli
Kostnaður	                     Yfirleitt lægri	                 Getur verið hærri
Sjálfstæð skjölun	             Takmörkuð	                       Betri
Áhrif bilana	                 API og Worker deila ferli	       Bilun í öðru þarf ekki að stöðva hitt
Viðhald	                       Færri rekstrareiningar	           Skýrari rekstraraðskilnaður
Uppfærslur	                   Gefin út saman	                   Hægt að gefa út sjálfstætt

2.b) Hvenær myndi ég velja hvort?
Ég myndi velja A ef:
- Gagnamagnið er hóflegt.
- Bakgrunnsvinnslan notar lítið af auðlindum.
- Við viljum lágmarka uppsetningu, kostnað og viðhald.
- Það er ásættanlegt að API og Worker séu uppfærð saman.

Ég myndi velja B ef:
- Bakgrunnsvinnslan er þung eða getur haft áhrif á svörun API.
- Við þurfum að skala vinnsluna sjálfstætt.
- Mismunandi teymi bera ábyrgð á API og vinnslu.
- Við viljum að API geti haldið áfram að starfa þótt Worker sé stöðvaður eða uppfærður.

Basic til að keyra:
1. dotnet run --project .\IntegrationLab.Api --urls http://localhost:5080
2. Invoke-RestMethod http://localhost:5080/health

Aðrar leiðir til að sjá svar:
1. Invoke-WebRequest — skoða hrátt svar:
(Invoke-WebRequest http://localhost:5080/transactions/1001).Content
{"id":1001,"amount":25000,"currency":"ISK","status":"Pending"}

2. ConvertTo-Json — breyta PowerShell-hlut aftur í JSON:
Invoke-RestMethod http://localhost:5080/transactions/1001 |
    ConvertTo-Json

3. curl.exe — sjá HTTP-svarið beint:
curl.exe -i http://localhost:5080/../..
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":1001,"amount":25000,"currency":"ISK","status":"Pending"}

Dæmi um hvernig hægt er að leita í postgres töflunum að data:
'SELECT * FROM "Transactions";' | docker exec -i integrationlab-postgres psql -U integrationlab -d integrationlab
