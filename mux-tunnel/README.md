# mux-tunnel

Reverse-ssh-Tunnel zum Connecten auf den RPI über einen externen virtuellen Server.
Geht der Tunnel verloren, wird erneut versucht, die Verbindung aufzubauen.

## Vorteile dieser Variante

* abgestimmt auf die eingeschränkte DS-Lite-Konnektivität
* IP oder DNS-Name des RPI muss nicht bekannt sein.
* ssh ist trivial zu konfigurieren gegenüber einem tun/tap-Tunnel.
* tun/tap muss auf dem VServer nicht vorhanden sein.
* Ressourcenschonend, auf das Wesentliche konzentriert.
* Nutzung von Linux-Standardtools.


## Clientseitige Konfiguration (Rasperry Pi)

* Paket tmux installieren
* Einen Benutzer namens `tunnel` anlegen
* Als Benutzer `tunnel` ein RSA-Schlüsselpaar erzeugen:
  * `$ ssh-keygen -t rsa -b 4096`
* Die Dateien `start-mux-tunnel.sh` und `keep-mux-tunnel.sh` in das Verzeichnis `/home/tunnel/` kopieren.
* Beide Dateien als Benutzer `tunnel` ausführbar machen.
* In `~/keep-mux-tunnel.sh` unter `server=` den Namen des externen V-Servers eintragen.

##### Mindest-Konfiguration des ssh-Clients in `/etc/ssh/ssh_config`:
```
Host *
    SendEnv LANG LC_*
    GSSAPIAuthentication yes
    ServerAliveInterval=180
    ServerAliveCountMax=3
```

## Serverseitige Konfiguration (externer V-Server)

* Einen Benutzer namens 'tunnel' anlegen.
* Öffentlichen RSA-Schlüssel des Rasperry Pi importieren.

##### Mindest-Konfiguration des ssh-Server in `/etc/ssh/sshd_config`:
```
GatewayPorts clientspecified
ClientAliveInterval 180
ClientAliveCountMax 3
Subsystem sftp /usr/lib/openssh/sftp-server
AcceptEnv LANG
AcceptEnv LC_*
PrintMotd yes
PermitRootLogin no
PermitEmptyPasswords no
ChallengeResponseAuthentication no
GSSAPIAuthentication no
HostbasedAuthentication no
KbdInteractiveAuthentication no
KerberosAuthentication no
PasswordAuthentication no
PubkeyAuthentication yes
UsePAM no
AllowUsers tunnel
```
##### ACHTUNG

In diesem Beispiel sind alle Authentifikationen mit Ausnahme von `PubkeyAuthentication` deaktiviert.
Ein Login, beispielsweise per Passwort, ist dann nicht mehr möglich.

## Grundlegende Benutzung

* Mit dem Befehl `~/start-mux-tunnel.sh` wird der ssh-Tunnel aufgebaut.
* Per `tmux attach` kann der Status des Tunnels live begutachtet werden.
* Innerhalb von tmux kann dann als Nutzer `tunnel` auf dem V-Server per ssh gearbeitet werden.
* Innerhalb von tmux kann das Terminal mit der Tastenfolge '[Ctrl-B] danach [D]' verlassen werden.
  Der Tunnel bleibt nach dem Verlassen mit dieser Tastenabfolge weiterhin bestehen.