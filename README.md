# Övning: Grunderna i docker

## Starta, stoppa containers

En docker-container är _ett körtillfälle_ av ett program, med utgångspunkt från filerna som finns i en docker-image. 

Vi provar att starta en ny docker-container som kör ett litet leksaksprogram, skrivet som ett `sh`-skript:    

    docker run -it alpine sh -c "while true; do echo hello from inside docker; sleep 1; done" 

Avbryt körningen med <kbd>Ctrl</kbd>+<kbd>C<kbd>

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
   
### Sista övning
   
Starta en docker-container från imagen `mbentley/figlet`. Ange kommandot `wohoo`.

### Städning
 
Se till att stoppa alla containers du har igång. `docker ps` ska vara tom. 

Första övningen klar! Wohoo!

## Servrar och portmappningar

Vi kan ju roa oss med andra docker-images än `alpine`, till exempel imagen `node:6.9.2`. Skrivformen `IMAGE:TAG` gör att vi väljer att köra en namngiven version av en image, `6.9.2` i det här fallet.

I nodejs kan vi skriva en enkel webbserver med följande program:

    require('http').createServer(function (req, res) { res.end('hejsan'); }).listen(3000);

Vi kan köra det under docker:

    docker run -d -p 8123:3000 node:6.9.2 node -e "require('http').createServer(function (req, res) { res.end('hejsan'); }).listen(3000);"
    
Prova nu att accessa http://127.0.0.1:8123, t.ex. i din browser. 

Vi har här använt en viktig feature i docker, portmappningar. En portmappning kopplar ihop ett portnummer på maskinen som kör docker på (även kallad docker host), till ett portnummer i din container. Det behövs eftersom varje docker-container kör med sin egen nätverks-stack, dvs sin egen rymd av portnummer etc och även en egen ip-adress. Via portmappningen kan du komma åt en port utifrån och på så sätt komma åt processer i en docker-container som lyssnar på nätverket. I exemplet ovan kopplade vi port 8123 på din maskin till port 3000 i docker-containern.

### Sista övning

Kör två containrar med nodejs-programmet ovan. Den ena kan svara "goddagens" och den andra "this is an ex-parrot", eller nåt annat lämpligt. Låt båda lyssna på port 3000 inuti docker men den ena vara tillgänglig på port 8124 och den andra på port 8125. Accessa dem var för sig i din browser. Stoppa dem båda.




