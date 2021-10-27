# hse21_hw1
Bioinformatics Minor HSE - Homework №1

# Майнор биоинформатика 2 год #

## Домашнее задание 1 ##

### Колчина Анастасия 4 группа ###

### 1. Список команд, выполненных на сервере ###

Создаем папочки:
```
$ mkdir hw1
$ cd hw1
$ mkdir reads
$ cd reads
```
Создаем символические ссылки на риды:
```
$ ls /usr/share/data-minor-bioinf/assembly/* | xargs -tI{} ln -s {}
```
С помощью команды seqtk выбираем случайно 5 миллионов чтений типа paired-end и 1.5 миллиона чтений типа mate-pairs:
```
$ seqtk sample -s126 oil_R1.fastq 5000000 > pe1.fastq
$ seqtk sample -s126 oil_R2.fastq 5000000 > pe2.fastq

$ seqtk sample -s126 oilMP_S4_L001_R1_001.fastq 1500000 > mp1.fastq
$ seqtk sample -s126 oilMP_S4_L001_R2_001.fastq 1500000 > mp2.fastq
```
Запустим fastqc, чтобы оценить качество исходных чтений:
```
$ mkdir fastqc
$ ls pe* mp* | xargs -P 4 -tI{} fastqc -o fastqc {}

$ mkdir multiqc
$ multiqc -o multiqc fastqc
```
Скачаем отчет с сервера на локальную машину. Для этого заходим в папку с ключом mykey для доступа на сервер и пишем следующую команду:
```
scp -P 5222 -i mykey akkolchina@92.242.58.92:~/hw1/reads/multiqc/multiqc_report.html ~/bio_minor/multiqc
```
Открываем html файл multiqc и видим следующую статистику:

![Screenshot (1346)](https://user-images.githubusercontent.com/60008375/138947442-96fc6b77-e9aa-4381-b304-a44551ac2f61.png)

![fastqc_sequence_counts_plot](https://user-images.githubusercontent.com/60008375/138947486-0230f4d8-b4d0-4521-a178-09d9cadcdcc8.png)

![fastqc_sequence_duplication_levels_plot](https://user-images.githubusercontent.com/60008375/138948152-2407e376-453c-4015-8cb3-0b5c66ce49d3.png)

![fastqc_per_sequence_quality_scores_plot](https://user-images.githubusercontent.com/60008375/138947688-0ea5e6ee-f1d7-40d1-a487-393e249a9ed5.png)

![fastqc_per_sequence_gc_content_plot](https://user-images.githubusercontent.com/60008375/138947709-128c96ea-9bb4-4a0b-a8fb-ded799c00df6.png)

![fastqc_per_base_sequence_quality_plot](https://user-images.githubusercontent.com/60008375/138947734-3fcf14c0-6682-4e3b-b61c-4642f5e57c63.png)

![fastqc_adapter_content_plot](https://user-images.githubusercontent.com/60008375/138947746-b447200f-7f49-420d-ad4e-b01abda53f7f.png)

С помощью программ platanus_trim и platanus_internal_trim подрезаем чтения по качеству и удаляем праймеры:
```
$ platanus_trim pe*
$ platanus_internal_trim mp*
```
Удаляем исходные ```fastq``` файлы (они больше не нужны):
```
$ rm pe1.fastq pe2.fastq mp1.fastq mp2.fastq
```
Запустим fastqc, чтобы оценить качество подрезанных чтений:
```
$ mkdir fastqc_trimmed
$ ls pe* mp* | xargs -P 4 -tI{} fastqc -o fastqc_trimmed {}

$ mkdir multiqc_trimmed
$ multiqc -o multiqc_trimmed fastqc_trimmed
```
Скачаем отчет с сервера на локальную машину. Для этого заходим в папку с ключом mykey для доступа на сервер и пишем следующую команду:
```
scp -P 5222 -i mykey akkolchina@92.242.58.92:~/hw1/reads/multiqc_trimmed/multiqc_report.html ~/bio_minor/multiqc_trimmed
```
Открываем html файл multiqc_trimmed и видим следующую статистику:

![Screenshot (1354)](https://user-images.githubusercontent.com/60008375/139093479-2b91fd2a-9896-401a-aeac-3ac294a75878.png)

![fastqc_sequence_counts_plot (1)](https://user-images.githubusercontent.com/60008375/139093565-62028ec9-ae44-4b77-9ed7-d1b36328281d.png)

![fastqc_sequence_duplication_levels_plot (1)](https://user-images.githubusercontent.com/60008375/139093608-44706eb2-37e9-4c29-ba9d-eaf8a38ab7b6.png)

![fastqc_per_sequence_quality_scores_plot (1)](https://user-images.githubusercontent.com/60008375/139093678-0a061594-0bee-4635-bbd9-5572d959c76f.png)

![fastqc_per_sequence_gc_content_plot (1)](https://user-images.githubusercontent.com/60008375/139093710-676941a9-9371-4af8-b364-7c7514c84c87.png)

![fastqc_per_base_sequence_quality_plot (1)](https://user-images.githubusercontent.com/60008375/139093756-e62564d1-407e-4969-9423-98495dcfb6d1.png)

![fastqc_adapter_content_plot (1)](https://user-images.githubusercontent.com/60008375/139093775-bd19a3ef-ed32-4d72-bfb2-73d59069d4ed.png)

С помощью программы ```platanus assemble``` соберем контиги из подрезанных чтений:
```
$ platanus assemble -o Poil -t 1 -m 16 -f pe1.fastq.trimmed pe2.fastq.trimmed 2>assemble.log
```
Далее проведем анализ полученных контигов:

С помощью программы ```platanus scaffold``` соберем скаффолды из контигов и подрезанных чтений:
```
$ platanus scaffold -o Poil -t 1 -c Poil_contig.fa -IP1 pe1.fastq.trimmed pe2.fastq.trimmed -OP2 mp1.fastq.int_trimmed mp2.fastq.int_trimmed 2> scaffold.log
```
С помощью программы ```platanus gap_close``` уменьшаем кол-во гэпов с помощью подрезанных чтений:
```
$ platanus gap_close -o Poil -t 1 -c Poil_scaffold.fa -IP1 pe1.fastq.trimmed pe2.fastq.trimmed -OP2 mp1.fastq.int_trimmed mp2.fastq.int_trimmed 2> gapclose.log
```
Удалим подрезанные ```fastq``` файлы (они больше не нужны):
```
rm pe* mp*
```
