# Övning: Grunderna i docker

## Övning 1: Starta, stoppa containers

En docker-container är _ett körtillfälle_ av ett program, med utgångspunkt från filerna som finns i en docker-image. 

Vi provar att starta en ny docker-container som kör ett litet leksaksprogram, skrivet som ett `sh`-skript:    

    docker run -it alpine sh -c "while true; do echo hello from inside docker; sleep 1; done" 

Avbryt körningen med <kbd>Ctrl</kbd>+<kbd>C</kbd>

Att starta en ny container görs alltså på följande form:

    docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Man anger alltså

* Eventuella options till `docker run`
* En docker-image, dvs den samling filer/program som ditt program utgörs av/har tillgång till.
* Du kan ange ett kommando, dvs. _vad_ du vill köra från programmen som finns i din image. Anger du inget körs standard-kommandot (som ibland inte är definierat).

Som options angav vi `-it` som innebär interaktivt läge, med output/input kopplat till din terminal. Oftast kör man docker som en bakgrundstjänst.
 
 Starta samma program, fast i bakgrunden istället:
 
    docker run -d alpine sh -c "while true; do echo hello from inside docker; sleep 1; done"

 Nu får du ingen output, utan istället skrivs ett _container-id_ ut, till exempel:
 
     4908f11d8dfed9488166c6f965c02e3c1401d74593c8db45a7d2b74957396813

Kontrollera vilka containers du har igång:

    docker ps 

Nu skrivs en lista av aktiva containers ut, till exempel:

     CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
    4908f11d8dfe        alpine              "sh -c 'while true..."   5 minutes ago       Up 5 seconds                            clever_shaw
 
Du kan nu prova att:

   * Inspektera en containers output med `docker logs CONTAINER`. Du kan ange både namn och id, från mitt exempel alltså antingen `4908f11d8dfe` eller `clever_shaw`.
   * Stoppa en container snällt med `docker stop CONTAINER` eller bryskt med `docker kill CONTAINER`
   * Starta fler kopior av vårt leksaksprogram, genom att köra `docker run -d` flera gånger.
   
### Slutkläm övning 1
   
Starta en docker-container från imagen `mbentley/figlet`. Ange kommandot `wohoo`.

### Städning
 
Se till att stoppa alla containers du har igång. `docker ps` ska vara tom. 

Första övningen klar! Wohoo!

## Övning 2: Servrar och portmappningar

Vi kan ju roa oss med andra docker-images än `alpine`, till exempel imagen `node:6.9.2`. Skrivformen `IMAGE:TAG` gör att vi väljer att köra en namngiven version av en image, `6.9.2` i det här fallet.

I nodejs kan vi skriva en enkel webbserver med följande program:

    require('http').createServer(function (req, res) { res.end('hejsan'); }).listen(3000);

Vi kan köra det under docker:

    docker run -d -p 8123:3000 node:6.9.2 node -e "require('http').createServer(function (req, res) { res.end('hejsan'); }).listen(3000);"
    
Prova nu att accessa http://127.0.0.1:8123, t.ex. i din browser. 

Vi har här använt en viktig feature i docker, portmappningar. En portmappning kopplar ihop ett portnummer på maskinen som kör docker på (även kallad docker host), till ett portnummer i din container. Det behövs eftersom varje docker-container kör med sin egen nätverks-stack, dvs sin egen rymd av portnummer etc och även en egen ip-adress. Via portmappningen kan du komma åt en port utifrån och på så sätt komma åt processer i en docker-container som lyssnar på nätverket. I exemplet ovan kopplade vi port 8123 på din maskin till port 3000 i docker-containern.

### Slutkläm övning 2

Kör två containrar med nodejs-programmet ovan. Den ena kan svara "goddagens" och den andra "this is an ex-parrot", eller nåt annat lämpligt. Låt båda lyssna på port 3000 inuti docker men den ena vara tillgänglig på port 8124 och den andra på port 8125. Accessa dem var för sig i din browser. Stoppa dem båda.




## Övning 3: Dockerfile och egna images

För att skapa egna images, till exempel för att paketera din egen kod, använder man en fil som kallas `Dockerfile` för att styra hur innehållet i en image byggs ihop. Här vill du alltså ange alla förutsättningar i form av program och bibliotek, med rätt versioner av allt, och kopiera in din egen kod.

Här ett exempel på en `Dockerfile`:

    FROM nodejs:6.9.2
    ADD app.js /
    CMD node app.js
    
Vad säger denna `Dockerfile` ? Jo:

* `FROM` används för att ange en annan image som utgångspunkt. Det håller din egen Dockerfile kortfattad, och alla måste ju börja nånstans, liksom. (`FROM scratch` börjar från ingenting).    
* `ADD` används för att kopiera in filer (eller `COPY`; skillnaden är subtil, kolla upp den själv).
* `CMD` anger programmet som standardmässigt ska startas från din image om inget annat angetts till `docker run`.

Skapa en tom katalog, `hello-docker`, i den lägger du en `Dockerfile` från ovan och en källkodsfil för ett nodejs-program, `app.js`.

Exempel på lämplig app.js:

    const http = require('http');

    const hostname = '127.0.0.1';
    const port = 3000;

    const server = http.createServer((req, res) => {
      res.end('Tjohoo allesammans\n');
    });

    server.listen(port, hostname, () => {
      console.log(`Server running at http://${hostname}:${port}/`);
    });


För att bygga en image från din Dockerfile kan du använda

    docker build -t hellonode .

`-t` används för att tagga (namnge) din image så du kan referera det namnet
vid `docker run`. Vi valde namnet hellonode.

* Bygg och kör din image
* Portmappa port 8126 till port 3000 i containern. Kör den i bakgrunden.

### Slutkläm övning 3

Övningen går ut på att köra den populära webbservern nginx i docker och serva statisk html.

Skapa en `Dockerfile` som baserar sig på imagen `nginx:1.11.10`. Skapa en fil `index.html` och fyll den med nån HTML, och se till att den kopieras till katalogen `/usr/share/nginx/html` med hjälp av Dockerfile-kommandot `COPY`. Kör din docker-image och portmappa port 8127 till port 80 i containern. Hämta sidan i valfri http-klient (kanske https://httpie.org/) eller din browser.


## Övning 4: Docker-compose

Ibland vill man kunna komponera ihop ett antal olika appar, till ett helt system
till exempel en webbserver och en databas.

För det kan man använda docker compose.

Gör getting-started övningen på https://docs.docker.com/compose/gettingstarted/
