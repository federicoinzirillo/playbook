---
title: Linux essenziale per cloud
sidebar_label: "0.6 Linux essenziale"
sidebar_position: 6
---

# Linux essenziale per cloud

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 0.6</span>
  <span>~9 min di lettura</span>
</div>

<p class="lesson-lead">Il cloud gira su Linux. Non devi diventare un amministratore di sistema — ma senza questo minimo, sei cieco su qualsiasi VM, container e log che ti troverai davanti.</p>

Hai imparato le risorse cloud, i livelli di astrazione, lo storage. Prima di procedere con networking, container e AWS in pratica, c'è una skill trasversale da cui non si scappa: leggere un filesystem Linux, capire cosa sta girando su una macchina, gestire i permessi, connetterti via SSH e leggere i log. Su AWS ogni VM è Linux (o Windows, ma la stragrande maggioranza è Linux). I container Docker girano su kernel Linux. I log che ti salveranno alle 3 di notte sono log Linux.

L'**idea in una frase**: il minimo Linux per il cloud non è saper installare pacchetti — è saper **navigare, leggere e diagnosticare** un sistema che qualcuno (spesso il provider) ha già configurato per te.

## Il filesystem: dove abita tutto

Linux organizza tutto in un unico albero che parte da `/` — la radice (*root*). Non ci sono lettere di drive come su Windows. Alcune directory sono standardizzate e le trovi su qualsiasi distribuzione:

| Directory | Cosa ci vive |
|---|---|
| `/etc` | File di configurazione di sistema (nginx.conf, sshd_config, cron, ecc.) |
| `/var/log` | Log di sistema e delle applicazioni — il tuo primo posto quando qualcosa va storto |
| `/home` | Directory home degli utenti (`/home/ubuntu`, `/home/ec2-user`) |
| `/tmp` | File temporanei, cancellati al reboot — non salvarci niente di importante |
| `/usr/bin`, `/usr/local/bin` | Dove stanno gli eseguibili installati |
| `/proc`, `/sys` | Filesystem virtuali: lo stato live del kernel, CPU, memoria, processi |

Per navigare:

```bash
pwd           # stampa la directory corrente
ls -la        # lista contenuto con permessi, owner, dimensione
cd /var/log   # cambia directory
cd ~          # vai alla home dell'utente corrente
cd -          # torna alla directory precedente
```

## Processi: cosa sta girando

Su una VM in produzione non sai sempre cosa gira. Questi sono i comandi che ti danno il quadro:

```bash
ps aux                   # tutti i processi in esecuzione (user, PID, CPU%, MEM%, comando)
top                      # vista live, aggiornata ogni secondo (q per uscire)
htop                     # come top ma più leggibile (se installato)
kill -9 1234             # termina il processo con PID 1234 forzatamente
systemctl status nginx   # stato di un servizio systemd (nginx, sshd, docker, ecc.)
systemctl restart nginx  # riavvia il servizio
journalctl -u nginx -f   # log live di un servizio systemd
```

Il segnale `-9` (SIGKILL) è il Kill nucleare: il processo viene terminato dal kernel senza possibilità di cleanup. Usalo solo quando un processo non risponde a `-15` (SIGTERM, il kill gentile che dà tempo di fare cleanup).

## Permessi: chi può fare cosa

I permessi Linux sono la base di ogni discussione sulla sicurezza di sistema. Ogni file e directory ha tre insiemi di permessi: per il **proprietario** (*user*), per il **gruppo** (*group*), per **tutti gli altri** (*others*).

```
-rwxr-xr-- 1 ubuntu devteam 4096 mag 20 14:30 deploy.sh
```

Leggi da sinistra: `-` = file (oppure `d` = directory), poi tre blocchi di tre caratteri:
- `rwx` — owner (ubuntu): read + write + execute
- `r-x` — group (devteam): read + execute, no write
- `r--` — others: solo read

`r`=4, `w`=2, `x`=1 — i permessi si esprimono anche in ottale:

```bash
chmod 755 deploy.sh    # owner: 7=rwx; group: 5=r-x; others: 5=r-x
chmod 600 chiave.pem   # owner: 6=rw-; group: 0=---; others: 0=---
chown ubuntu:devteam deploy.sh  # cambia owner e group
```

`chmod 600` su una chiave SSH privata è quasi obbligatorio: SSH si rifiuta di usare file con permessi troppo aperti. È il tuo primo incontro con il principio del least privilege applicato ai file.

## SSH: connettersi alle VM

**SSH** (*Secure Shell*) è il protocollo per connettersi a macchine remote in modo sicuro. Su AWS ogni EC2 instance si raggiunge via SSH.

Il meccanismo base: crei una **coppia di chiavi** (pubblica + privata). La chiave pubblica va sulla VM (nel file `~/.ssh/authorized_keys` dell'utente); la chiave privata rimane solo sul tuo laptop. Quando ti connetti, SSH dimostra che hai la chiave privata corrispondente senza trasmetterla mai. Niente password.

```bash
# Connessione base
ssh -i ~/.ssh/mia-chiave.pem ubuntu@ec2-3-120-45-67.eu-west-1.compute.amazonaws.com

# Con alias in ~/.ssh/config (molto più pratico)
Host mia-ec2
  HostName ec2-3-120-45-67.eu-west-1.compute.amazonaws.com
  User ubuntu
  IdentityFile ~/.ssh/mia-chiave.pem

ssh mia-ec2   # da qui in poi basta questo
```

Alcune cose da sapere:
- **Il file `.pem` non si condivide mai** e deve avere permessi 400 o 600 (`chmod 400 mia-chiave.pem`).
- L'utente predefinito su EC2 varia per immagine: `ubuntu` per Ubuntu, `ec2-user` per Amazon Linux, `admin` per Debian.
- Se SSH dice "WARNING: UNPROTECTED PRIVATE KEY FILE", la chiave ha permessi troppo aperti. Fai `chmod 600` e riprova.

Per copiare file da/verso una VM remota:

```bash
scp -i ~/.ssh/mia-chiave.pem file.txt ubuntu@ec2-3-120-45-67.eu-west-1.compute.amazonaws.com:/home/ubuntu/
rsync -avz -e "ssh -i ~/.ssh/mia-chiave.pem" ./locale/ ubuntu@IP:/home/ubuntu/remota/
```

## Log: dove leggere quando qualcosa va storto

I log sono la finestra sullo stato del sistema. Sul cloud, il primo posto dove guardare quando qualcosa smette di funzionare:

```bash
# Log di sistema (systemd / kernel)
journalctl -xe              # log di sistema recenti con contesto
journalctl -u nginx --since "1 hour ago"  # log specifico di nginx dell'ultima ora
journalctl -f               # log live (segue in tempo reale)

# Log classici in /var/log
tail -f /var/log/nginx/access.log   # log accessi nginx, aggiornato in tempo reale
tail -100 /var/log/nginx/error.log  # ultime 100 righe del log errori
grep "ERROR" /var/log/app.log       # cerca ERROR nel log
grep "ERROR" /var/log/app.log | wc -l  # conta quante righe contengono ERROR

# Combinazioni utili
cat /var/log/syslog | grep -i "oom"     # cerca Out-Of-Memory killer
dmesg | tail -50                         # messaggi del kernel (hardware, boot, OOM)
```

Tre comandi da tenere a mente: `tail -f` per seguire un log live, `grep` per filtrare per pattern, `|` (pipe) per combinare comandi.

Su AWS il monitoring serio si fa con **Amazon CloudWatch** — log centralizzati, metriche, alert — ma saper leggere i log di sistema direttamente è la skill di fallback quando CloudWatch non dice abbastanza o quando sei in debug su una VM fresca.

<details>
<summary>Qualche comando utile in più</summary>

**Dischi e filesystem:**
```bash
df -h          # spazio usato su ogni filesystem montato
du -sh /var/*  # spazio usato da ogni directory in /var
lsblk          # elenco dei block device (dischi, partizioni)
```

**Rete:**
```bash
ip addr show          # indirizzi IP delle interfacce di rete
curl -I https://example.com  # header HTTP di una risposta (utile per debug)
wget -q -O - http://169.254.169.254/latest/meta-data/  # metadata EC2 (solo dentro AWS)
```

**Variabili d'ambiente:**
```bash
env              # stampa tutte le variabili d'ambiente
export VAR=val   # imposta una variabile per la sessione corrente
echo $PATH       # vedi il PATH corrente
```

**`/etc/hosts` e DNS:**
```bash
cat /etc/hosts    # mappature locali hostname → IP
nslookup google.com  # risoluzione DNS
dig +short google.com  # risoluzione DNS, output compatto
```
</details>

## Cosa non è

| Il pensiero sbagliato | Come stanno le cose |
|---|---|
| "Con il cloud non ho bisogno di Linux" | I container girano su Linux, le VM sono Linux, i log che leggi sono Linux. Il serverless nasconde il runtime, ma il debugging richiede comunque queste skill. |
| "SSH serve solo per VM: con i container non mi serve" | Con i container hai `docker exec -it container_id bash` per aprire una shell dentro un container in esecuzione — stesso concetto, stessa skill. |
| "I permessi Linux sono roba da sysadmin" | Il principio del least privilege (base di IAM cloud) è lo stesso dei permessi Unix. Capire uno aiuta a capire l'altro. |
| "I log sono in `/var/log` e basta" | Su sistemi moderni con systemd i log stanno nel journal (binario), accessibile con `journalctl`. `/var/log` spesso ha solo link o file scritti da applicazioni che non usano systemd. |

## Verifica di comprensione

> Rispondi a memoria. Le risposte incerte rivedile domani.

1. Dove trovi i log di sistema su una VM Linux?
2. Cosa significa `chmod 755`? E `chmod 600`?
3. Perché SSH non accetta un file `.pem` con permessi `644`?
4. Come visualizzi i log live di nginx su una VM con systemd?
5. Cosa fa `grep "ERROR" app.log | wc -l`?
6. Qual è la differenza tra `kill -9` e `kill -15`?
7. *(anticipazione)* Se la tua VM EC2 non è raggiungibile via SSH, quali altre informazioni puoi cercare prima di ricominciare da capo?

---

## Glossario della pagina

- **journalctl**: strumento per leggere i log del journal systemd; usato su distribuzioni Linux moderne.
- **Least privilege**: principio di sicurezza: dare a ogni entità (utente, processo, servizio) solo i permessi strettamente necessari.
- **Permessi Unix**: sistema di controllo accesso a tre livelli (owner/group/others) con tre bit (read/write/execute) per livello.
- **PID** (*Process Identifier*): numero univoco assegnato a ogni processo in esecuzione su Linux.
- **SSH** (*Secure Shell*): protocollo crittografato per connessione remota a macchine Linux; usa coppie di chiavi asimmetriche.
- **Systemd**: init system e gestore di servizi usato sulla maggior parte delle distribuzioni Linux moderne (incluse le AMI Amazon Linux e Ubuntu su AWS).

## Per approfondire

- `man bash`, `man ssh`, `man chmod` — le pagine man sono il riferimento definitivo, disponibili su qualsiasi sistema Linux.
- "The Linux Command Line" di William Shotts — disponibile liberamente online (`linuxcommand.org`); il riferimento per chi parte da zero.
- "SSH Essentials" su DigitalOcean Tutorials (`digitalocean.com/community/tutorials`) — ottima guida pratica SSH, verificata e aggiornata.

## Prossima lezione

Hai le fondamenta: cos'è il cloud, come si sceglie il livello di astrazione, quali risorse esistono, dove vivono geograficamente, come si persistono i dati, e come muoversi su Linux. La **Parte 1** entra nel networking — la parte dove i full-stack hanno i buchi più grossi, e dove si fanno i danni più difficili da trovare.
