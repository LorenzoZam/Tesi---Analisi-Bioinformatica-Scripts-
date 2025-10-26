# Pipeline — Analisi Bioinformatica (Script raccolti)

Questo repository raccoglie una serie di script Python utili per processare risultati di Mass Spectrometry e generare liste pronte per analisi funzionali e visualizzazioni (preranked, filtraggio conta[...]
Di seguito trovi una versione ristrutturata e più leggibile della guida all'uso, con i mini-capitoli evidenziati in grassetto e titoli più grandi.

---

## Indice
- [Panoramica](#panoramica)
- [Requisiti](#requisiti)
- [Struttura e scopo degli script](#struttura-e-scopo-degli-script)
  - **1 — GENERAZIONE LISTA DA FILE MASS SPEC**
  - **2 — RIMOZIONE CONTAMINANTI**
  - **3 — GENERAZIONE LISTE UNIVOCHE**
  - **4 — ORA su GO + GRAFICO DOTPLOT**
  - **5 — GRAFICO VENN**
- [Formati input / output](#formati-input--output)
- [Contatti / Licenza](#contatti--licenza)

---

## Panoramica
Questo set di script è pensato per essere usato passo‑passo:
1. Generare una lista "preranked" a partire da file mass‑spec grezzi.  
2. Rimuovere contaminanti comuni.  
3. Ottenere liste specifiche per condizione.  
4. Eseguire analisi di arricchimento funzionale (ORA) e plottare i risultati.  
5. Generare diagrammi di Venn per confronti tra insiemi.

---

## Requisiti
- Python 3.8+  
- Pacchetti Python principali: `pandas`, `tkinter`, `pathlib`, `numpy` 
- Per plotting / ORA (script 4/5): `gseapy`, `matplotlib_venn`, `seaborn`
- Nota: gli script attuali usano `tkinter` per aprire dialog di selezione file.

---

## Struttura e scopo degli script

### **1 — GENERAZIONE LISTA DA FILE MASS SPEC**
- Scopo: creare una lista preranked (GENE[TAB]SCORE) a partire da file mass spec (.txt, .csv, .xlsx).
- Algoritmo (riepilogo):
  - Legge il file riga per riga, individua righe con header di proteina (pattern `tr|` o `sp|` nella prima colonna) ed estrae `GN=<gene>`.
  - Dalla riga prende la sequenza peptidica (assume colonna index 2) e l'E-value (assume colonna index 4).
  - Per ogni peptide prende il miglior (minore) E-value, calcola `-log10(best_evalue)` e somma per gene/proteina.
  - Applica un limite minimo di E-value a `1e-12` per evitare `-log10(0)`.
  - Output: file tab-separated con righe `GENE<TAB>SCORE` ordinate per score decrescente (score con 4 decimali).
- Input supportati: `.xlsx`, `.csv`, `.txt` (tab).
- Nota pratica: se le colonne peptide/E-value sono in posizioni diverse, modificare gli indici nello script.

---

### **2 — RIMOZIONE CONTAMINANTI**
- Scopo: filtrare via proteine comunemente considerate contaminanti (actine, tubuline, cheratine, emoglobine, albumina, immunoglobuline, HSP, ribosomiali, ecc.).
- Come funziona:
  - Accetta file `.txt` o `.xls/.xlsx` con colonne `GENE` / `SCORE` (o due colonne Excel).
  - Normalizza i nomi dei geni in uppercase e confronta con una lista interna di contaminanti.
  - Produce due file nella stessa cartella:
    - `<input_stem>_filter.txt` → lista pulita (proteine mantenute);
    - `<input_stem>_filter_out.txt` → lista dei contaminanti rimossi.
- Output: file testo con `GENE[TAB]SCORE` (o solo `GENE` se score non presente).

---

### **3 — GENERAZIONE LISTE UNIVOCHE**
- Scopo: identificare proteine/gene specifiche di ciascuna lista tra un insieme di liste (es. condizioni).
- Come funziona:
  - Carica più file selezionati (.txt o .xlsx) trasformandoli in dict `{GENE: score}` (se manca lo score, usa `0`).
  - Per ogni lista calcola i geni presenti solo in quella lista (non nelle altre).
  - Salva per ogni input un file chiamato `<input_stem>_specific.txt` con le proteine specifiche ordinate per score decrescente.
- Nota: i geni vengono convertiti in uppercase per confronto.

---

### **4 — ORA su GO + GRAFICO DOTPLOT**
- Scopo: eseguire Over‑Representation Analysis (GO) e produrre un dotplot.
  - Mapping gene IDs (simboli o UniProt) a database GO tramite `gseapy`.
  - Scelta del background (universe): es. intero proteoma murino o lista definita dall'utente.
  - Output: tabelle con termini significativi (p-value, FDR) + figura dotplot (`png`/`pdf`).
- Verificare lo script per parametri configurabili: numero di termini mostrati, ontologie (BP/MF/CC), FDR threshold.

---

### **5 — GRAFICO VENN**
- Scopo: creare diagrammi di Venn per confrontare insiemi/proteine tra condizioni.
- Input tipico: file con liste univoche (output del passo 3) o più file ognuno rappresentante un set.
- Pacchetti comuni: `matplotlib_venn` (Python) o alternative in R (`VennDiagram`, `eulerr`).
- Output: immagine (`.png`/`.pdf`) e (opzionale) tabelle con le intersezioni.

---

## Formati input / output consigliati
- File mass-spec: `.txt` (tab), `.csv`, `.xlsx` — assicurarsi di conoscere quali colonne contengono peptide e E-value.  
- File preranked / output script 1: tab-separated text file con `GENE<TAB>SCORE` senza intestazione.  
- File filtrati: stesso formato (nomi in uppercase consigliati).  
- File per Venn/ORA: liste di geni (una per riga) o file con due colonne (`GENE, SCORE`) se richiesto dallo script.

---

## Informazioni aggiuntive
- La lista di contaminanti è inclusa direttamente nello script 2: modificabile per esperimento e specie in esame.  
- Gli script attualmente sono interattivi (usano dialog di `tkinter`) — è relativamente semplice adattarli per esecuzione da CLI con parametri.

---

## Contatti / Licenza
- Autore: Lorenzo Zammariello (account github: LorenzoZam) 
- Licenza MIT, disponibile nella repository
