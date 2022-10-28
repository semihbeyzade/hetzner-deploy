# Deployment

Es muss ein account bei Hetzner erstellt werden: https://www.hetzner.com/

Navigiere zu https://console.hetzner.cloud/projects

Neues Projekt erstellen und einen server erstellen:
- Location wählen (in deutschland)
- Betriebssystem: Ubuntu 20.04
- Typ Standart & kleinster Rechner für 3,81€ im Monat
- wählt einen server-name (ganz unten)

Installiere dir das VS-Code Plugin "Remote - SSH". Danach erscheint ein grüner Button links unten in VS-Code. Bitte klicken:

- Wähle: "Verbindung mit Host herstellen"
- Wähle "# Neue Verbindung herstellen"
- Tippe "ssh root@5.75.159.83 -A" (Bitte eigene IP)
- Wähle Speicherort aus. Das erste was vorgeschlagen wird

Öffne ein Terminal auf deinem Rechner
- Tippe "ssh root@5.75.159.83" (Bitte eigene IP)
- Gib dein Passwort ein und ändere Das Passwort

- Klicker erneut auf den grünen button links unten
- Wähle: "Verbindung mit Host herstellen"
- Wähle deine IP aus --> es öffnet sich ein neues VS-Code Fenster
- Fingerabdruck bestätigen ("Weiter" klicken)
- Passwort eingeben (dein neu erstelltes)
- Öffne das Terminal

## Server einrichten

- `sudo apt-get update`
- `sudo apt-get install curl`
- `sudo apt-get install build-essential`
- `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.2/install.sh | bash`
- `export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion`
- `nvm install 16`

## Projekt einrichten

- `mkdir /var/www`
- `mkdir /var/www/project`
- In VS-Code File-System: Klicke "Ordner öffnen"
- Tippe "/var/www/project/" in die Eingabe und klicke OK

Es öffnet sich VS-Code erneut in dem Folder "/var/www/project"

## Projekt von Github ziehen

- `git clone https://github.com/D04-1/devq-mini-project.git .` (bitte eure github url angeben)

Wenn ihr zwei repos habt sehen die befehle so aus:
- `git clone https://github.com/D04-1/project-1.git`
- `git clone https://github.com/D04-1/project-2.git`


Nicht vergessen die `.env` Datei im Backend zu erstellen

## MongoDB installieren
(https://www.digitalocean.com/community/tutorials/how-to-install-mongodb-on-ubuntu-20-04)

- `curl -fsSL https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -`
- `echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list`
- `sudo apt update`
- `sudo apt install mongodb-org`
- `sudo systemctl start mongod.service`
- `sudo systemctl status mongod` -> schau ob mongodb läuft

## Projekt starten

- Installiere alle Packages in frontend und backend (npm install)
- starte backend & frontend
- projekt läuft unter [ip]:[port]

- Tausche alle URLs aus welche auf localhost zeigen im frontend. diese sollen auf die IP zeigen
- Im Backend muss die Cors-Config auch auf die IP zeigen

## Apache Installieren

- `sudo apt update && sudo apt -y upgrade`
- `sudo apt install -y apache2`
- `sudo a2enmod proxy`
- `sudo a2enmod proxy_http`
- `sudo service apache2 restart`
- `code /etc/apache2/sites-available/project.conf`

Es öffnet sich ein neues file. hier bitte folgendes eintrage:

```
<VirtualHost *:80>
    ServerName devq

    ProxyRequests Off
    ProxyPreserveHost On
    ProxyVia Full

    <Proxy *>
        Require all granted
    </Proxy>

    ProxyPass / http://127.0.0.1:3000/
    ProxyPassReverse / http://127.0.0.1:3000/
</VirtualHost>
```

- `a2ensite project.conf`
- `a2dissite 000-default`
- `a2enmod proxy proxy_http rewrite headers expires`
- `systemctl restart apache2`

## App über gemeinsammen port ausliefern

- gehe in das frontend projekt
- `npm run build`
- schiebe den erstellten folder (build) in das backend und nimm ihn in die gitignore auf

bearbeite die app.js im backend:

```javascript
app.use(express.static(require('path').join(__dirname, "build")))
```

Das liefert die fertige React-App unter port 80 (default-port) aus. Damit alles funktioniert muss noch folgendes getan werden:

- in der backend `.env` müsssen wir den port 3000 angeben (der von apache weitergeleitet wird)
- wir müssen die domain aus allen frontend-requests enfernen (z.b `http://5.75.159.83:3001/user` --> `/user`)
- cors aus der app.js im backend entfernen (brauchen wir nicht mehr)
- frontend app muss wieder gebaut werden (`npm run build`) und der build folder ins backend kopiert werden

## App persistieren

- `npm install -g pm2`
- im backend-folder: `pm2 start app.js`

App kann gestoppt werden mit `pm2 stop app.js` (im backend-folder)

## Branch

speichert alle Änderungen in einem eigenen branch `live` damit eure dev-config nicht davon betroffe ist


# Neue Änderungen deployen

Deine Änderungen müssen in den main-branch kommen und nach github gepushed werden

- `git checkout main`
- `git pull` --> hol dir die Änderungen
- `git checkout live`
- `git merge main`
- frontend: `npm run build` --> kopieren den build-folder ins backend
- backend: `pm2 stop app.js`
- backend `pm2 start app.js`
