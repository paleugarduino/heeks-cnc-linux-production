# HeeksCAD & HeeksCNC - Linux Production Edition

Questo repository contiene una versione ottimizzata e corretta di HeeksCAD e HeeksCNC,<br>
specificamente modificata per garantire stabilità, reattività e affidabilità in ambienti di produzione industriale<br>
su sistemi Linux (Debian/Ubuntu).

Il codice originale di Dan Heeks soffriva di colli di bottiglia critici dovuti all'interfaccia grafica GTK2<br>
e alla gestione asincrona del server grafico X11, che causavano freeze e crash distruttivi quando si lavorava<br>
con file G-code di grandi dimensioni in officina. Questa edizione risolve questi problemi alla radice.

---

## 🛠️ Bug Corretti in questa Edizione

### 1. Rimozione del Blocco su File G-code Mastodontici (HeeksCNC)
 Problema: Il coloratore sintattico del testo (`FormatText`) generava migliaia di tag grafici GTK2.<br>
Con file G-code da migliaia di righe, la CPU schizzava al 100% congelando l'applicazione per minuti.
 Soluzione: Disattivata la formattazione dei colori pesante su Linux.<br>
La generazione e il caricamento del codice finale sono ora istantanei (frazioni di secondo anche su file enormi).

### 2. Risoluzione del "Freeze della Finestra Grigia" in Sovrascrittura (HeeksCAD)
 Problema: Quando si salvava il G-code finale sovrascrivendo il pre-gcode, l'applicazione controllava<br>
continuamente gli appunti di sistema tramite X11 dentro un ciclo di Idle.<br>
Questo innescava un deadlock con la finestra modale di salvataggio, bloccando l'interfaccia su un rettangolo grigio vuoto.
 Soluzione: Cortocorticuitata la funzione `IsPasteReady()`.<br>
Il controllo ossessivo della clipboard su X11 è stato disattivato, eliminando definitivamente lo stallo grafico.

### 3. Scudo contro i Crash del Mouse nel Viewport 3D (HeeksCAD)
 Problema: Se si muoveva il mouse sul display 3D subito dopo la rigenerazione di un percorso utensile,<br>
il programma cercava gli ID grafici dei vecchi vettori ormai rimossi dalla memoria.<br>
Un comando `assert()` rigido causava il crash immediato del software (`SIGABRT`).
 Soluzione: Sostituto l'assert in `Index.h` con un controllo di sicurezza tollerante.<br>
Se l'oggetto non esiste più, il programma restituisce `NULL` e continua a girare senza interrompere il lavoro dell'operatore.

---

## 📦 Come Compilare i Pacchetti .deb (Ambiente Chroot Bionic)

Per rigenerare i pacchetti puliti senza sporcare il sistema operativo ospite,<br>
esegui i seguenti comandi da utente normale all'interno del tuo ambiente di sviluppo:

```bash
mkdir -p ~/heeks-dev && cd ~/heeks-dev

git clone [https://github.com/paleugarduino/heeks-cnc-linux-production.git](https://github.com/paleugarduino/heeks-cnc-linux-production.git)

```

### Compilazione Libarea

```bash
cd ~/heeks-dev/heeks-cnc-linux-production/libarea

debian/rules clean

PATH=$PATH:/usr/sbin:/sbin \
LD_LIBRARY_PATH=/lib/x86_64-linux-gnu:/usr/lib/x86_64-linux-gnu \
dpkg-buildpackage -us -uc -b -j$(nproc)

```

### Compilazione HeeksCAD

```bash
cd ~/heeks-dev/heeks-cnc-linux-production/heekscad

debian/rules clean

PATH=$PATH:/usr/sbin:/sbin \
LD_LIBRARY_PATH=/lib/x86_64-linux-gnu:/usr/lib/x86_64-linux-gnu \
dpkg-buildpackage -us -uc -b -j$(nproc)

```

### Compilazione HeeksCNC

```bash
cd ~/heeks-dev/heeks-cnc-linux-production/heekscnc

debian/rules clean

PATH=$PATH:/usr/sbin:/sbin \
LD_LIBRARY_PATH=/lib/x86_64-linux-gnu:/usr/lib/x86_64-linux-gnu \
dpkg-buildpackage -us -uc -b -j$(nproc)

```

### 🚀 Installazione di tutti i pacchetti compilati

Accedi come utente root ed esegui i seguenti comandi per installare le dipendenze e i pacchetti generati:

```bash
apt-get install python-ocl

cd ~/heeks-dev/heeks-cnc-linux-production

dpkg -i .deb

apt-get --fix-broken install

```

---

## ⚖️ Licenza

Questo progetto è rilasciato sotto la BSD 3-Clause License (New BSD License), mantenendo la licenza originale stabilita da Dan Heeks.


Consulta il file `LICENSE` per maggiori dettagli.

