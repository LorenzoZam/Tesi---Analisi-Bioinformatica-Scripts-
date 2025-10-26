```markdown
# Pipeline — Analisi Bioinformatica (Script raccolti)

Questo repository raccoglie una serie di script Python utili per processare risultati di Mass Spectrometry e produrre liste pronte per analisi funzionali e visualizzazioni (preranked, filtraggio contaminanti, liste specifiche, ORA/plot e Venn). Di seguito trovi una guida dettagliata per usare gli script presenti e il formato di input/output atteso.

Indice
- Panoramica
- Requisiti
- Struttura e scopo degli script
  - 1 - GENERAZIONE LISTA DA FILE MASS SPEC
  - 2- RIMOZIONE CONTAMINANTI
  - 3 - GENERAZIONE LISTE UNIVOCHE
  - 4- ORA su GO + GRAFICO DOTPLOT
  - 5- GRAFICO VENN
- Formati input / output
- Esempio di flusso di lavoro
- Suggerimenti pratici
- Ambiente consigliato

Panoramica
Questo set di script è pensato per essere usato passo‑passo:
1) generare una lista "preranked" a partire da file mass‑spec grezzi;  
2) rimuovere contaminanti comuni;  
3) ottenere liste specifiche per condizione;  
4) eseguire analisi di arricchimento funzionale (ORA) e plottare i risultati;  
5) generare diagrammi di Venn per confronti tra insiemi.

Requisiti
- Python 3.8+  
- Pacchetti Python utilizzati (minimo, per gli script già presenti): pandas, tkinter  
- Per plotting/ORA (script 4/5) è stato utilizzato gseapy.

Note su GUI: gli script attuali usano tkinter per aprire file dialog 


Struttura e scopo degli script (dettagli tecnici rilevati)

1) 1 - GENERAZIONE LISTA DA FILE MASS SPEC
- Scopo: creare una lista preranked (GENE[TAB]SCORE) a partire da file mass spec (.txt, .csv, .xlsx).
- Algoritmo implementato:
  - Lo script legge il file riga per riga e cerca righe con header di proteina (pattern "tr|" o "sp|" nella prima colonna). Da queste righe tenta di estrarre GN=<gene>.
  - Per le righe associate prende la sequenza peptidica (assume colonna index 2) e l'E-value (assume colonna index 4).
  - Per ogni peptide si prende il miglior (minore) E-value, si calcola -log10(best_evalue) e si sommano i valori per ogni gene/proteina.
  - Applica un limite minimo dell'E-value a 1e-12 (per evitare -log10 di 0).
  - Output: file di testo/tab separato con righe "GENE<TAB>SCORE" ordinato in ordine decrescente di score (score stampato con 4 decimali).
- Input supportati: .xlsx, .csv, .txt (tab).
- Note pratiche:
  - Se il tuo file ha le colonne peptide/E-value in posizioni diverse, modifica gli indici (row[2] e row[4]) nello script.
  - Gestione dei decimali: valori con virgola come separatore decimale ("1,2e-3") sono convertiti correttamente.
  - Lo script attuale è interattivo (chiede file da aprire e posizione di salvataggio tramite dialog).

2) 2- RIMOZIONE CONTAMINANTI
- Scopo: filtrare via proteine comunemente considerate contaminanti (actine, tubuline, cheratine, emoglobine, albumina, immunoglobuline, heat shock proteins, ribosomiali, ecc.).
- Come funziona:
  - Accetta file .txt o .xls/.xlsx con colonne (GENE /tab SCORE oppure due colonne Excel).
  - Normalizza i nomi dei geni a uppercase e confronta con una lista interna di contaminanti (lista codificata nello script).
  - Produce due file nella stessa cartella: 
    - <input_stem>_filter.txt → lista pulita (proteine mantenute);
    - <input_stem>_filter_out.txt → lista dei contaminanti rimossi.
- Output: file testo con per riga GENE[TAB]SCORE (se score presente), altrimenti solo GENE.

3) 3 - GENERAZIONE LISTE UNIVOCHE
- Scopo: identificare proteine/gene specifiche di ciascuna lista tra un insieme di liste (es. condizioni).
- Come funziona:
  - Carica più file selezionati (.txt o .xlsx) trasformandoli in dict {GENE: score} (se manca lo score, usa 0).
  - Per ogni lista calcola i geni presenti solo in quella lista (non nelle altre).
  - Salva per ogni input un file chiamato <input_stem>_specific.txt con le proteine specifiche ordinate per score decrescente.
- Input: multipli file .txt o .xlsx selezionati tramite dialog.
- Nota: i geni vengono convertiti in uppercase per confronto.

4) 4- ORA su GO + GRAFICO DOTPLOT
- Scopo atteso (da nome file): eseguire Over‑Representation Analysis (GO) e produrre un dotplot.
    - mapping gene IDs (simboli o UniProt) su database GO (mediante gseapy)
    - scelta del background (universe): in questo caso è stato utilizzato l'intero proteoma murino.
    - output: tabelle con termini significativi (p-value, FDR) + figura dotplot (png/pdf)
  - Verificare lo script per eventuali parametri (numero di termini mostrati, ontologie considerate: BP/MF/CC).

5) 5- GRAFICO VENN
- Scopo atteso (da nome): creare diagrammi di Venn per confrontare insiemi/proteine tra condizioni.
- Implementazione/consigli:
  - Input tipico: file con liste univoche (output del passo 3), o più file ognuno rappresentante un set.
  - Pacchetti comuni: matplotlib_venn (Python) o VennDiagram/eulerr (R).
  - Output: immagine (.png/.pdf) + (opzionale) tabelle con le intersezioni.

Formati input / output consigliati
- File mass-spec: .txt (tab), .csv, .xlsx. Assicurarsi di conoscere quali colonne contengono peptide e E-value.
- File preranked/output script 1: tab-separated text file con "GENE<TAB>SCORE", senza intestazione.
- File filtrati: same format (uppercase per i nomi dei geni).
- File per Venn/ORA: liste di geni (una per riga) o file con due colonne (GENE, SCORE) se richiesto dallo script.

Esempio di flusso rapido (interattivo con dialog)
1. Lanciare lo script 1 e selezionare file mass-spec:
   - python "1 - GENERAZIONE LISTA DA FILE MASS SPEC"
   - Salvare la lista preranked (es. samples_preranked.txt)
2. Lanciare lo script 2 e selezionare samples_preranked.txt
   - Otterrai samples_preranked_filter.txt e samples_preranked_filter_out.txt
3. Lanciare lo script 3 e selezionare uno o più file filter.txt (es. per confrontare condizioni)
   - Otterrai file *_specific.txt per ciascun input
4. Usare gli output del passo 3 come input per ORA (script 4) e per i Venn (script 5).



Informazioni aggiuntive
- La lista di contaminanti è inclusa direttamente nello script 2: modificabile per esperimento e specie in esame.

Contatti / Licenza
- Autore: vedi cronologia commit del repository.
- Aggiungi qui la licenza desiderata (es. MIT) se vuoi distribuire pubblicamente gli script.

```
