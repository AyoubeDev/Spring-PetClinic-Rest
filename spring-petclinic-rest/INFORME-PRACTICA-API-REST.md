# Informe de la pràctica de proves API REST

## 1. Objectiu

Dissenyar i executar proves d’integració per a Spring PetClinic REST utilitzant Postman i Newman.

## 2. Preparació

L’API s’executa a:

```text
http://localhost:9966/petclinic
```

Per iniciar-la a Windows:

```powershell
$env:JAVA_HOME='C:\Program Files\Java\jdk-27'
.\mvnw.cmd spring-boot:run
```

L’entorn de Postman utilitza:

```text
baseUrl = http://localhost:9966/petclinic
```

## 3. Estructura de la col·lecció

La col·lecció està organitzada en els dominis següents:

- `01 - First Visit`: Owner -> Pet -> Visit.
- `02 - Returning Customer`: actualització d’Owner i Pet.
- `03 - Owners CRUD complet`: CRUD complet d’Owners.
- `04 - Pets CRUD complet`: CRUD complet de Pets.
- `05 - Vets`: consultes de veterinaris.
- `06 - Negative cases`: proves negatives.
- `07 - Visits CRUD complet`: CRUD de Visits.
- `08 - Data-driven`: petició preparada per utilitzar dades d’iteració.

Les variables `ownerId` i `petId` es guarden a partir de les respostes de les peticions POST i es reutilitzen en les peticions següents.

## 4. Operacions i codis HTTP

| Operació | Resultat esperat |
|---|---:|
| Crear Owner, Pet o Visit | 201 |
| Consultar un recurs existent | 200 |
| Actualitzar Owner o Pet | 204 |
| Eliminar Owner o Pet | 204 |
| Consultar un recurs eliminat | 404 |
| Owner sense `firstName` | 400 |
| Owner o Pet inexistent | 404 |

Les assertions es basen en el comportament real observat a l’API.

## 5. Validacions implementades

La col·lecció comprova:

- El codi HTTP.
- La resposta JSON quan correspon.
- Els identificadors i els tipus de dades.
- La relació entre Owner i Pet.
- La persistència dels canvis després d’un PUT.
- El comportament després d’un DELETE.
- Un temps de resposta inferior a 2000 ms.
- El contracte JSON Schema bàsic per a Owners.

## 6. Casos negatius i valors límit

S’inclouen els casos negatius següents:

1. Crear un Owner sense `firstName`.
2. Consultar un Owner inexistent.
3. Crear un Pet amb un Owner inexistent.
4. Crear una Visit amb un Pet inexistent.

També s’ha comprovat la validació del telèfon de l’Owner:

- 10 dígits: vàlid.
- 9 o 11 dígits: l’API retorna 500.
- Lletres: l’API retorna 400.

Els resultats s’han documentat sense inventar codis HTTP diferents dels observats.

## 7. Incidències trobades

### JAVA_HOME

Maven no s’iniciava perquè `JAVA_HOME` no estava definit. Es va solucionar establint la ruta del JDK per a la sessió de PowerShell.

### Compilació antiga

En crear un Pet, el servidor retornava un error 500 relacionat amb `PetAgeValidation`. Tot i que el fitxer existia, s’estaven utilitzant classes compilades antigues. Es va solucionar amb:

```powershell
.\mvnw.cmd clean spring-boot:run
```

### Variables de Postman

Quan no se seleccionava l’entorn `Petclinic Env`, les variables no es resolien. La col·lecció requereix seleccionar aquest entorn abans d’executar les peticions.

### Ubicació incorrecta de les peticions

Algunes peticions afegides manualment havien quedat dins del camp `response`, que només conté exemples de resposta. Es van recol·locar dins de `item` perquè Postman les pogués executar.

## 8. Data-Driven Testing

Es va preparar el fitxer `csv/owners.csv` amb casos vàlids i un cas invàlid:

```csv
firstName,lastName,address,city,telephone,statusEsperado
Ayoube,Serra,Carrer Major 1,Vic,6345678901,201
Joan,Perez,Carrer Nou 2,Barcelona,6345678901,201
Carles,Garcia,Carrer de la llibertat 3,Vic,6345678901,201
,Garcia,Carrer Invalid 3,Vic,6345678901,400
Seiki,,Carrer Sobirà 6,Ceuta,6345678901,400
```

La col·lecció utilitza `pm.iterationData` per llegir les dades i comprovar el codi esperat de cada iteració.

El compte de Postman utilitzat mostra que els datasets del Runner requereixen un pla de pagament. Per aquest motiu, es conserven les evidències disponibles i es documenta aquesta limitació. La mateixa execució es pot reproduir gratuïtament amb Newman.

## 9. Execució amb Newman

L’informe HTML es genera a:

`reports/petclinic-report.html`

La col·lecció final conté 36 peticions i 83 assertions. L’informe s’ha de regenerar amb la col·lecció final perquè inclogui les carpetes de Visits i Data-driven.

Comanda:

```powershell
npx.cmd --yes -p newman -p newman-reporter-htmlextra newman run `
  .\src\test\postman\petclinic-nonregressiontests.postman_collection.json `
  -e .\src\test\postman\petclinic-env.postman_environment.json `
  --reporters cli,htmlextra `
  --reporter-htmlextra-export .\src\test\postman\reports\petclinic-report.html
```

## 10. Ús assistit d’IA

S’ha utilitzat la IA com a suport per proposar casos addicionals sobre `POST /api/owners`. Les propostes s’han revisat manualment segons les validacions reals del servei.

| Cas proposat | Dades | Resultat esperat | Decisió |
|---|---|---:|---|
| Owner vàlid | Tots els camps complets i telèfon de 10 dígits | 201 | Acceptat |
| Falta `firstName` | `firstName` buit | 400 | Acceptat |
| Falta `address` | `address` buit | 400 | Acceptat |
| Telèfon amb 9 dígits | `telephone = 612345678` | 400 segons la validació esperada | Revisat: l’API retorna 500 |
| Telèfon amb lletres | `telephone = abcdefghij` | 400 | Acceptat |

El cas del telèfon de 9 dígits no es va convertir en una assertion de 400 perquè l’API real retorna 500. Aquesta diferència es documenta com una incidència en lloc d’inventar un contracte.

Prompt utilitzat:

```text
A partir de l’endpoint POST /api/owners i de les seves validacions reals,
proposa cinc casos de prova positius, negatius i de valors límit.
Indica les dades, el codi HTTP esperat i les assertions.
No inventis regles que no apareguin en el contracte.
```

Colateralment de tota la correció ortigrafica i en estructura gramatical.

## 11. Conclusions

Crec que els meus objectius com a programador coincideixen amb els de la pràctica. Poder desenvolupar una bateria de proves, enviar peticions a una API i validar-ne les respostes és una competència molt útil. Tot i així, considero que es podrien trobar altres rutes d’aprenentatge més eficients. Un projecte d’un mes no sempre produeix els mateixos resultats d’aprenentatge que diversos projectes petits treballats durant el mateix període.

Pel que fa als requisits, considero que hi ha algunes ambigüitats, especialment tenint en compte que és el primer cop que treballem amb APIs. Per exemple, no es fixen els codis HTTP esperats, sinó que es demana descobrir-los. Aquesta tasca pot requerir molt de temps i pot generar resultats diferents segons la interpretació de cada persona. Tampoc queda completament clar si s’han de provar tots els endpoints disponibles o només els necessaris per complir els requisits mínims.

En el cas dels Pets, es combinen endpoints anidats, com ara `/owners/{ownerId}/pets/{petId}`, amb endpoints directes com `/pets/{petId}`. Per a una persona que treballa per primera vegada amb una API, aquesta diferència pot resultar confusa i pot fer necessari consultar documentació i eines de suport aliénes.

També hi ha una certa falta de concreció en la gestió de variables. No s’especifica clarament quan s’han d’utilitzar variables d’entorn, de col·lecció o globals. Es proporciona l’exemple `pm.collectionVariables.set`, però l’environment també conté variables. A més, no s’explica com gestionar possibles conflictes o dades residuals quan la col·lecció s’executa diverses vegades.

Durant les proves també s’han observat respostes de l’API que no coincidien amb les esperades inicialment. Per exemple, algunes dades incorrectes retornaven un error 500 en lloc d’un 400. Aquest tipus de comportament complica l’aprenentatge, perquè obliga a diferenciar entre el contracte esperat i el comportament real de la implementació.

Pel que fa a Postman, considero que és una eina amb moltes funcionalitats i utilitats rellevants en un entorn professional. La seva interfície és bastant intuïtiva i permet organitzar peticions, variables, assertions i informes d’una manera pràctica.

En resum, considero que aquesta pràctica és útil per aprofundir en la relació entre petició, resposta i validació. Tot i les ambigüitats i la complexitat inicial, el projecte permet entendre millor com es dissenya una bateria de proves d’API i com s’automatitza la seva execució.

psdt: el punt 9 executar test untest amb un dataset csv, no s'ha pogut executar degut a que em demanaba fer un pagament. totes les proves a la carpeta de assets dins de l'arrel. (spring-petclinic-rest\assets). per el demès. tot completat. en la mateixa carpeta estan totes les altres evidencies en captures.