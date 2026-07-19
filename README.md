```markdown
# HeeksCAD & HeeksCNC - Linux Production Edition

Questo repository contiene una versione ottimizzata e corretta di  HeeksCAD e HeeksCNC,
specificamente modificata per garantire stabilità, reattività e affidabilità in ambienti di produzione industriale
su sistemi Linux (Debian/Ubuntu).

Il codice originale di Dan Heeks soffriva di colli di bottiglia critici dovuti all'interfaccia grafica GTK2
e alla gestione asincrona del server grafico X11, che causavano freeze e crash distruttivi quando si lavorava
con file G-code di grandi dimensioni in officina. Questa edizione risolve questi problemi alla radice.

## 🛠️ Bug Corretti in questa Edizione

### 1. Rimozione del Blocco su File G-code Mastodontici (HeeksCNC)
Problema: Il coloratore sintattico del testo (`FormatText`) generava migliaia di tag grafici GTK2.
Con file G-code da migliaia di righe, la CPU schizzava al 100% congelando l'applicazione per minuti.
Soluzione: Disattivata la formattazione dei colori pesante su Linux. La generazione e
il caricamento del codice finale sono ora **istantanei** (frazioni di secondo anche su file enormi).

### 2. Risoluzione del "Freeze della Finestra Grigia" in Sovrascrittura (HeeksCAD)
Problema: Quando si salvava il G-code finale sovrascrivendo il pre-gcode, l'applicazione controllava
continuamente gli appunti di sistema tramite X11 dentro un ciclo di Idle.
Questo innescava un deadlock con la finestra modale di salvataggio, bloccando l'interfaccia su un rettangolo grigio vuoto.
Soluzione: Cortocircuitata la funzione `IsPasteReady()`.
Il controllo ossessivo della clipboard su X11 è stato disattivato, eliminando definitivamente lo stallo grafico.

### 3. Scudo contro i Crash del Mouse nel Viewport 3D (HeeksCAD)
Problema: Se si muoveva il mouse sul display 3D subito dopo la rigenerazione di un percorso utensile,
il programma cercava gli ID grafici dei vecchi vettori ormai rimossi dalla memoria. Un comando `assert()`
rigido causava il crash immediato del software (`SIGABRT`).
Soluzione: Sostituito l'assert in `Index.h` con un controllo di sicurezza tollerante.
Se l'oggetto non esiste più, il programma restituisce `NULL` e continua a girare senza interrompere il lavoro dell'operatore.

## 📦 Come Compilare i Pacchetti .deb (Ambiente Chroot Bionic)

Per rigenerare i pacchetti puliti senza sporcare il sistema operativo ospite, esegui i seguenti comandi da utente normale all'interno del tuo ambiente di sviluppo:

```
### Compilazione Libarea
```bash
cd ~/heeks-dev/libarea
debian/rules clean
PATH=$PATH:/usr/sbin:/sbin LD_LIBRARY_PATH=/lib/x86_64-linux-gnu:/usr/lib/x86_64-linux-gnu dpkg-buildpackage -us -uc -b -j$(nproc)

```
### Compilazione HeeksCAD
```bash
cd ~/heeks-dev/heekscad
debian/rules clean
PATH=$PATH:/usr/sbin:/sbin LD_LIBRARY_PATH=/lib/x86_64-linux-gnu:/usr/lib/x86_64-linux-gnu dpkg-buildpackage -us -uc -b -j$(nproc)

```
### Compilazione HeeksCNC
```bash
cd ~/heeks-dev/heekscnc
debian/rules clean
PATH=$PATH:/usr/sbin:/sbin LD_LIBRARY_PATH=/lib/x86_64-linux-gnu:/usr/lib/x86_64-linux-gnu dpkg-buildpackage -us -uc -b -j$(nproc)

```
### Installazione di tutti i paccketti compilati 
```bash
Da root dare i seguenti comandi:

apt-get install python-ocl

cd ~/heeks-dev
dpkg -i *.deb

apt-get apt --fix-broken install
```

## ⚖️ Licenza

Questo progetto è rilasciato sotto la **BSD 3-Clause License** (New BSD License), mantenendo la licenza originale stabilita da Dan Heeks. 
Consulta il file `LICENSE` per maggiori dettagli.

```
