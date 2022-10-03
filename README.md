# hse22_hw1

Создадаю символьные ссылки на файлы:

```bash
ls /usr/share/data-minor-bioinf/assembly/* | xargs -tI{} ln -s {}
```

Выбираю случайно 5 миллионов чтений типа paired-end и 1.5 миллиона чтений типа mate-pairs с помощью команды seqtk

```bash
SEED=408
seqtk sample -s$SEED oil_R1.fastq 5000000 > paired_end_R1.fastq
seqtk sample -s$SEED oil_R2.fastq 5000000 > paired_end_R2.fastq
seqtk sample -s$SEED oilMP_S4_L001_R1_001.fastq 1500000 > mate_pairs_R1.fastq
seqtk sample -s$SEED oilMP_S4_L001_R2_001.fastq 1500000 > mate_pairs_R2.fastq
```

Оценю качество исходных чтений с помощью fastQC (создаю директории и заускаю fastQC для файлов, полученных выше):

```bash
mkdir fastqc
mkdir multiqc
ls paired_end_R* mate_pairs_R* | xargs -tI{} fastqc -o fastqc {}
```

Собираю отчёт с помощью multiQC:

```bash
multiqc -o multiqc fastqc
```

![image](https://user-images.githubusercontent.com/50082204/193601037-8a725411-e29c-4296-a43b-cbcbe13de2ec.png)
![image](https://user-images.githubusercontent.com/50082204/193601831-35c7b1d2-dbe6-4149-a498-0ec3706930af.png)
![image](https://user-images.githubusercontent.com/50082204/193601566-28b5ce94-b9e2-49af-90de-6d30373b5c14.png)


Подрезаю чтения по качеству:

```bash
platanus_trim paired_end_R*
platanus_internal_trim mate_pairs_R*
```


Оцениваю качество подрезанных данных с помощью fastQC:

```bash
mkdir fastqc_trimmed
mkdir multiqc_trimmed
ls paired_end_R* mate_pairs_R*| xargs -tI{} fastqc -o fastqc_trimmed {}
```

Собираю отчёт с помощью multiQC:

```bash
multiqc -o multiqc_trimmed fastqc_trimmed
```

![image](https://user-images.githubusercontent.com/50082204/193602170-5929b659-bb69-4a74-b86f-c596d4528c70.png)
![image](https://user-images.githubusercontent.com/50082204/193602271-9d1d0401-3000-4a66-99ad-22df65d7c632.png)
![image](https://user-images.githubusercontent.com/50082204/193602388-efe730fd-b782-4976-b2e6-a71766c96461.png)


Собираю контиги из подрезанных чтений с помощью “platanus assemble”:

```bash
time platanus assemble -o Poil -f paired_end_R1.fastq.trimmed paired_end_R2.fastq.trimmed 2> contigues.log
```

Анализ полученных контигов:

![image](https://user-images.githubusercontent.com/50082204/193627051-e594b493-dd6b-4af6-ae8c-e3acd9e05f29.png)


Собираю скаффолды из контигов и подрезанных чтений с помощью “platanus scaffold”:

```bash
time platanus scaffold -o Poil -c Poil_contig.fa -IP1 paired_end_R1.fastq.trimmed paired_end_R2.fastq.trimmed -OP2 mate_pairs_R1.fastq.int_trimmed mate_pairs_R2.fastq.int_trimmed 2> scaffolds.log
```

Анализ полученных скаффолдов:

![image](https://user-images.githubusercontent.com/50082204/193629341-7c245ffe-14af-414a-81e8-7085ea7fffcd.png)


Уменьшаю количество гэпов с помощью подрезанных чтений программой “platanus gap_close”:

```bash
time platanus gap_close -o Poil -c Poil_scaffold.fa -IP1 paired_end_R1.fastq.trimmed paired_end_R2.fastq.trimmed -OP2 mate_pairs_R1.fastq.int_trimmed mate_pairs_R2.fastq.int_trimmed 2> gapclose.log
```

Информация по гэпам после уменьшения их количества:

![image](https://user-images.githubusercontent.com/50082204/193629512-9090df84-ff65-4bbe-8fcd-cbf4dbeb78a9.png)


Удаляю исходные и с подрезанными чтениями файлы *.fastq:

```bash
rm paired_end_R*.fastq mate_pairs_R*.fastq
```
