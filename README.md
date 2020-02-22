# Szybkie pliki referencyjne do dockera na VPS

1. Zakładam że masz domenę i masz dostęp do konfiguracji rekordów typu **A** tej domeny
    1. Zanim zaczniesz robić coś dalej - sprawdź to i ustaw jeden tak by kierował na IP twojego vps.
2. Skonfiguruj sobie rzeczy w pliku `DB/docker-compose.yaml` 
    1. Nazwa kontenera dla porządku
    2. Hasła, nazwy i startową schemę (`MYSQL_DATABASE=`)
    3. Przekierowanie portu (o ile chcesz mieć więcej niż jeden kontener z bazą)
    4. Podsieć dockera (o ile zamierzasz cokolwiek z nią robić)
3. Baza *NIE* jest za proxy - jest dostępna pod IP vps (oczywiście o ile wcześniej nie usunąłeś expose i prawodłowo przekierowałeś porty)
    1. Start bazy - w jej katalogu <br> `docker-compose up -d`  
5. Skonfiguruj (o ile potrzebujesz) rzeczy w pliku `ngixProxy/docker-compose.yaml`
    1. Ale wydaje mi się że standardy są dla początkujących całkiem ok.
    2. Można dodać email (ale nie trzeba)
        ```
          letsencrypt:
             environment:
               - DEFAULT_EMAIL=twoj.email@twoja.domena.pl
        ```
    3.  Start - w katalogu ngixProxy <br> `docker-compose up -d`
6. Kontener do aplikacji spring boot - założenia
    1. Aplikacja działa pod java 1.8 (alpine) 
        1. Jesli nie dostosuj linię <br>
        `FROM openjdk:8-jdk-alpine` <br>
        w pliku Dockerfile <br>  https://hub.docker.com/_/openjdk?tab=tags znajdź swoją wersję
    2. Zakładam że Twoja aplikacja pluje logami do pliku (lub plików) zlokalizowanych w <br>
        `/var/log/twoja-app/` <br>
        i ten folder wystawiam na zewnątrz kontenera po to by był do nich łatwy dostęp
    3. Dla celów tego sposobu konteneryzacji appka musi działać na porcie **80** (i nie ma co się martwić, nie będzie kolizji na serwerze, proxy się wszystkim zajmie)
    4. Dostęp do bazy danych masz skonfigurowany po IP VPS (chyba że dla bazy też masz nazwę domenową)
7. Konfiguracja konteneryzacji
    1. Do katalogu SpringBoot dorzucasz swojego fatjara springboot
    2. W Dockerfile poprawiasz linię <br>
    `ARG JAR_FILE=twoja-appka.jar` <br>
    tak żeby twoja-appka.jar została zastąpiona nazwą fatjara
    3. W `docker-compose.yaml` ustawiasz parametry
        1. TWOJA_NAZWA_KONTENERA (to taki dla elegancji)
        2. adres.appki.w.twojej.domenie - to jest to co ustawiłeś w punkcie 1, pod tym adresem ma być dostępna appka.
8. Uruchamianie
    1. W katalogu SpringBoot masz 3 pliki - jar i dwa dockerowe.
    2. `docker-compose build up -d`
Jeśli wszystko poszło dobrze - masz dostęp do aplikacji pod adresem domenowym.


# Konfiguracja firewall pod Ubuntu i Docker

I'd like to thank to https://stackoverflow.com/users/553374/feng for tutorial:
https://stackoverflow.com/questions/30383845/what-is-the-best-practice-of-docker-ufw-under-ubuntu
- na stronie należy znaleźć `Solving UFW and Docker issues` dodać podane linie do odpowiedniego pliku
a potem wystawić na świat te usługi które są za dockerem - np. port 80 i 443 i 3306

```
sudo ufw allow ssh
sudo ufw route allow proto tcp from any to any port 80
sudo ufw route allow proto udp from any to any port 80
sudo ufw route allow proto tcp from any to any port 443
sudo ufw route allow proto udp from any to any port 443
sudo ufw route allow proto tcp from any to any port 3306
sudo ufw route allow proto udp from any to any port 3306
```