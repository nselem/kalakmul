![Better Lab](betterlab.jpg)


---
# Pre-procesamiento
---
La forma mÃ¡s habitual para importar datos en qiime2 es en formato *.fastq*. El formato fastq incluye cuatro lÃ­neas para cada read:

1. Identificador y datos de secuenciaciÃ³n
2. Secuencia
3. Signo '+'
4. Datos de calidad para cada base dentro de la secuencia

Ejemplo:

@JSS004-MS-Arch517F-MS-909R::M02696:148:000000000-BRYV4:1:2114:23978:14060
AAAACAAAGGGAAGCGTTGATCACCTCCAGGGCCTTATCGAAGTCGCTGAAAGGCTCCAGGATCAGAACCGGGCCGAAGACTTCGTTGTCGTAGACGCGGCAGTCCCGATCGACATTTTCGA
+
IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIFIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII

<br> 

En este caso los datos proporcionados tienen por separado un archivo con los identificadores y las secuencias (.fna) y otro con los identificadores y los datos de calidad de las secuencias (.qual). Hay que trabajar entonces en conseguir un formato final *.fastq*. Para ello, lo mÃ¡s sencillo es usar la paqueterÃ­a de BioPython y la funciÃ³n *seqIO* y aplicar el siguiente script para el archivo en formato *.fna*. Es importante tener en una misma carpeta el *.fna* y el archivo *.qual* para poder aplicar el script, pues detectarÃ¡ el archivo *.qual* de manera automÃ¡tica. 

Script:

>#!/usr/bin/env python
>import sys
>from Bio import SeqIO
>from Bio.SeqIO.QualityIO import PairedFastaQualIterator
>#!/usr/bin/env python
>import sys
>from Bio import SeqIO
>from Bio.SeqIO.QualityIO import PairedFastaQualIterator
>
>#Takes a FASTA file, which must have a corresponding .qual file,
>#and makes a single FASTQ file.
>
>if len(sys.argv) == 1:
>        print ("Please specify a  single FASTA file to convert.")
>        sys.exit()
>
>filetoload = sys.argv[1]
>basename = filetoload
>
>#Chop the extension to get names for output files
>if basename.find(".") != -1:
>        basename = '.'.join(basename.split(".")[:-1])
>
>try:
>        fastafile = open(filetoload)
>        qualfile = open(basename + ".qual")
>except IOError:
>        print ("Either the file cannot be opened or there is no >corresponding")
>        print ("quality file (" + basename +".qual)")
>        sys.exit()
>
>rec_iter = PairedFastaQualIterator(fastafile,qualfile)
>
>SeqIO.write(rec_iter, open(basename + ".fastq", "w"), "fastq")

Una vez obtenido el archivo *.fastq* se debe aplicar otro script para separarlo en archivos separados con el nombre de las distintas muestras JSS1, JSS2, ..., JSS17. 

<br>

Script:

`grep JSS Varela_5580B.fastq | cut -d'-' -f1 | sort | uniq | while read line ; do grep -A 3 $line Varela_5580B.fastq > $line.fastq ; done`

Se obtendrÃ¡n entonces archivos con las secuencias correspondientes a cada muestra. Los datos estÃ¡n listos para la importaciÃ³n a qiime2.

- [X] Preprocesamiento
- [ ] ImportaciÃ³n
- [ ] AnÃ¡lisis de calidad
- [ ] Denoising
- [ ] Diversidad 
- [ ] TaxonomÃ­a


---
# ImportaciÃ³n
---
Para comenzar con la importaciÃ³n debemos de cargar los paths absolutos de cada uno de los archivos en un archivo de texto separado por tabuladores que QIIME2 llama *manifest* creado en la terminal. En este caso yo utilicÃ© **vim** pero puede hacerse con cat o con nano y debe tener la siguiente estructura:

> manifest-arquea-33.txt

<br>

|**Sample_ID** | ---tab--- | **forward-absolute-filepath**|---tab---| **reverse-absolute-filepath**|
|-------------|---------|------------------|----------|------------|
|sample_1 | ---tab--- | /PWD/home/Varela_X/@JSS1_R1.fastq|---tab---|/PWD/home/Varela_X/@JSS1_R2.fastq|
|sample_2 | ---tab--- | /PWD/home/Varela_X/@JSS2_R1.fastq|---tab---|/PWD/home/Varela_X/@JSS2_R2.fastq|
|sample_3 | ---tab--- | /PWD/home/Varela_X/@JSS3_R1.fastq|---tab---|/PWD/home/Varela_X/@JSS3_R2.fastq|
|sample_4 | ---tab--- | /PWD/home/Varela_X/@JSS4_R1.fastq|---tab---|/PWD/home/Varela_X/@JSS4_R2.fastq|
   .               .                        .             .              .
   .               .                        .             .              .
   .               .                        .             .              .
sample_17 | ---tab--- | PWD/home/Varela_X/@JSS17_R1.fastq|---tab---|PWD/home/Varela_X/@JSS17_R2.fastq|


#### Una vez que tenemos el archivo de texto...

`conda activate qiime2`

(considerando que ya estÃ¡ instalado y en una environment variable en la terminal)
Utilizamos el siguiente comando para importaciÃ³n considerando importar de manera separada a los distintos dominios, Ã©sto para facilitar el posterior anÃ¡lisis: 

`qiime tools import --type 'SampleData[PairedEndSequencesWithQuality]' --input-path manifest-arquea-33.txt  --output-path arquea-demux.qza --input-format PairedEndFastqManifestPhred33V2`

- [X] Preprocesamiento
- [X] ImportaciÃ³n
- [ ] AnÃ¡lisis de calidad
- [ ] Denoising
- [ ] Diversidad 
- [ ] TaxonomÃ­a

---

El output serÃ¡ el archivo en formato *.qza* que convertiremos en *.qzv* para visualizarlo despuÃ©s:

`qiime demux summarize --i-data arquea-demux.qza --o-visualization /denoising/arquea-demux.qzv`

<br>

- [X] Preprocesamiento
- [X] ImportaciÃ³n
- [X] AnÃ¡lisis de calidad
- [ ] Denoising
- [ ] Diversidad 
- [ ] TaxonomÃ­a

![title](quality.png)

---
DespuÃ©s de analizarse la calidad de las muestras de arqueas, fue notable el largo irregular de los reads (~500) como se muestra en la figura. Para asegurarse de la longitud de los reads en las muestras, se aplicÃ³ el siguiente script:

`grep -v '>' JSSx.fastq |while read line; do num=$(echo $line | wc -c); echo $num; done | sort -n| tail`

<br>

Se obtuvieron un promedio de secuencias de ~316, confirmando la longitud a pesar de que la grÃ¡fica de calidad muestra una calidad mÃ¡xima de ~500.

---
# denoising
---
DADA2 es el algoritmo encargado de un filtro secundario en el que se cortan los extremos o inicios de los reads. En este caso se cortaron los reads a la longitud de 300 pb.

En el siguiente link se encuentra informaciÃ³n sobre todos los parÃ¡metros disponibles para el anÃ¡lisis con DADA2 en QIIME2:
[DADA2-qiime2](https://docs.qiime2.org/2020.11/plugins/available/dada2/)

`qiime dada2 denoise-single --i-demultiplexed-seqs archea-demux.qza --p-trim-left-f 0 --p-trim-left-r 0 --p-trunc-len-f 300 --p-trunc-len-r 300 --o-representative-sequences ../denoising/arc-rep-seqs.qza --o-table ../denoising/arc-table.qza --o-denoising-stats ../denoising/arc-stats.qza`

`qiime metadata tabulate --m-input-file arc-stats.qza --o-visualization arc-stats.qzv`

`qiime feature-table summarize --i-table arc-table.qza --o-visualization arc-table.qzv`

`qiime feature-table tabulate-seqs --i-data arc-rep-seqs.qza --o-visualization arc-rep-seqs.qzv`

---

- [X] Preprocesamiento
- [X] ImportaciÃ³n
- [X] AnÃ¡lisis de calidad
- [X] Denoising
- [ ] Diversidad 
- [ ] TaxonomÃ­a

![title](samples.jpg)
arc-table.qzv

![title](denoised.png)
arc-rep-seqs.qzv

![title](rep-seqs.png)
arc-stats.qzv 

---
# Diversidad 
---

`qiime phylogeny align-to-tree-mafft-fasttree --i-sequences arc-rep-seqs.qza --o-alignment aligned-arc-rep-seqs.qza --o-masked-alignment masked-aligned-arc-rep-seqs.qza --o-tree unrooted-tree.qza --o-rooted-tree rooted-tree.qza`

`qiime diversity core-metrics-phylogenetic --i-phylogeny ../phylogeny/rooted-tree.qza --i-table
 ../denoising/arc-table.qza --p-sampling 5000 --m-metadata-file ..
/../metadataJan.txt --output-dir core-metrics-results`

[Diversity-explanation-forum](https://forum.qiime2.org/t/alpha-and-beta-diversity-explanations-and-commands/2282)

`qiime diversity alpha-group-significance --i-alpha-diversity core-metrics-results/faith_pd_vector.qza --m-metadata-file metadata-arch.txt --o-visualization core-metrics-results/faith-pd-group-significance.qzv`

`qiime diversity alpha-group-significance --i-alpha-diversity core-metrics-results/evenness_vector.qza --m-metadata-file metadata-arch.txt --o-visualization core-metrics-results/evenness-group-significance.qzv`

`qiime diversity alpha-group-significance --i-alpha-diversity chao1_vector.qza --m-metadata-fil
e metadata-arch.txt --o-visualization core-metrics-results/chao1-g
roup-significance.qzv`

`qiime diversity alpha-group-significance --i-alpha-diversity simpson_vector.qza --m-metadata-f
ile metadata-arch.txt --o-visualization core-metrics-results/simps
on-group-significance.qzv`

`qiime diversity alpha-group-significance --i-alpha-diversity shannon_vector.qza --m-metadata-f
ile metadata-arch.txt --o-visualization core-metrics-results/shann
on-group-significance.qzv`

`qiime diversity beta-group-significance --i-distance-matrix core-metrics-results/unweighted_unifrac_distance_matrix.qza --m-metadata-file metadata-arch.txt --m-metadata-column habitat --o-visualization core-metrics-results/unweighted-unifrac-habitat-significance.qzv --p-pairwise`

`qiime diversity beta-group-significance --i-distance-matrix core-metrics-results/unweighted_unifrac_distance_matrix.qza --m-metadata-file metadata-arch.txt --m-metadata-column sample --o-visualization core-metrics-results/unweighted-unifrac-sample-significance.qzv --p-pairwise`

### NOTA: Emperor no salen por no tener columnas con carcteres solamente numÃ©ricos en metadata file. 

---

- [X] Preprocesamiento
- [X] ImportaciÃ³n
- [X] AnÃ¡lisis de calidad
- [X] Denoising
- [X] Diversidad 
- [ ] TaxonomÃ­a


# TaxonomÃ­a

[Classifier-forum](https://docs.qiime2.org/2021.4/tutorials/feature-classifier/)

Puedes hacer o no extraccion de secuencias si cuentas con las secuencias de los primers a la hora de entrenar el clasificador. Si no se tienen: 

`qiime feature-classifier classify-sklearn --i-classifier ~classifier/silva-138-99-nb-classifier.qza --i-reads ../denoising/arc-rep-seqs.qza --o-classification arc-taxonomy.qza`

`qiime metadata tabulate --m-input-file arc-taxonomy.qza --o-visualization taxonomy.qzv`

`qiime taxa barplot --i-table ../denoising/arc-table.qza --i-taxonomy taxonomy.qza --m-metadata-file metadata-arc.txt --o-visualization taxa-bar-plot.qzv`

---
- [X] Preprocesamiento
- [X] ImportaciÃ³n
- [X] AnÃ¡lisis de calidad
- [X] Denoising
- [X] Diversidad 
- [X] TaxonomÃ­a


![alt text](taxonomia.png)
