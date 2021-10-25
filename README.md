# hse21_hw1

1. Сначала надо создать папку "hw" в которой будут находиться все файлы и подключить линки к ней:
```
mkdir hw
cd hw
mkdir 1
cd 1
ls -1 /usr/share/data-minor-bioinf/assembly/* | xargs -tI{} ln -s {}
```

2. Производим случайные чтения и удаляем ненужные файлы:
```
seqtk sample -s1115 oil_R1.fastq 5000000 > pe_R1.fastq
seqtk sample -s1115 oil_R2.fastq 5000000 > pe_R2.fastq
seqtk sample -s1115 oilMP_S4_L001_R1_001.fastq 1500000 > mp_R1.fastq
seqtk sample -s1115 oilMP_S4_L001_R2_001.fastq 1500000 > mp_R2.fastq

rm -r oil_R1.fastq
rm -r oil_R2.fastq
rm -r oilMP_S4_L001_R1_001.fastq
rm -r oilMP_S4_L001_R2_001.fastq
```

3. Оцениваем качество исходных чтений с помощью fastQC и multiQC:
```
mkdir fastqc
ls *.fastq | xargs -P 4 -tI{} fastqc -o fastqc {}

mkdir multiqc
multiqc -o multiqc fastqc
```

Все файлы скачивали с помощью уточки. Кря (Cyberduck)

4. Обрезаем чтение по качеству и убираем праймеры: 
```
platanus_trim pe_R1.fastq pe_R2.fastq 
platanus_internal_trim mp_R1.fastq mp_R2.fastq  
```
Удаляем ненужные файлы
```
ls -1 *.fastq | xargs -tI{} rm -r {}
```
5. Оцениваем качество новых обрезанных чтений с помощью fastQC и multiQC:
```
mkdir trimmed_fastq
mv -v *trimmed trimmed_fastq/
mkdir trimmed_fastqc
ls trimmed_fastq/* | xargs -P 4 -tI{} fastqc -o trimmed_fastqc {}
mkdir trimmed_multiqc
multiqc -o trimmed_multiqc trimmed_fastqc
```
6. Сравниваем полученные результаты:
До 
<img width="1104" alt="Снимок экрана 2021-10-25 в 23 59 33" src="https://user-images.githubusercontent.com/93148512/138770095-3548767e-2366-4906-b9a8-87351aa67093.png">

После: 
<img width="1052" alt="Снимок экрана 2021-10-26 в 00 01 58" src="https://user-images.githubusercontent.com/93148512/138770351-f70b1a17-f710-40fc-8ef4-7cd94852328e.png">

До:
<img width="1053" alt="Снимок экрана 2021-10-26 в 00 02 42" src="https://user-images.githubusercontent.com/93148512/138770447-948a67e0-9b9d-45a6-a592-7f9e208584fa.png">

После:
<img width="1064" alt="Снимок экрана 2021-10-26 в 00 03 06" src="https://user-images.githubusercontent.com/93148512/138770499-73b74a9b-d45c-4a58-b898-1d3101c6284c.png">

До:
<img width="1067" alt="Снимок экрана 2021-10-26 в 00 03 35" src="https://user-images.githubusercontent.com/93148512/138770555-6072c264-3998-4d45-b554-bf5e87974c68.png">

После:
<img width="1046" alt="Снимок экрана 2021-10-26 в 00 04 11" src="https://user-images.githubusercontent.com/93148512/138770628-1df21305-c493-4012-81e8-536050c93e3d.png">

7. Cобираем контиги из подрезанных чтений и анализируем в Jupiter Notebook (результаты см. в блокноте):
```
time platanus assemble -o Poil -f trimmed_fastq/pe_R1.fastq.trimmed trimmed_fastq/pe_R2.fastq.trimmed 2> assemble.log
```
<img width="331" alt="Снимок экрана 2021-10-26 в 00 20 40" src="https://user-images.githubusercontent.com/93148512/138772754-0d85a1ee-b18a-4ff0-8e6c-0cf80d0d5dce.png">

8. Собираем скаффолды из контигов и подрезанных чтений и анализируем результаты в блокноте: 
```
time platanus scaffold -o Poil -c Poil_contig.fa -IP1 trimmed_fastq/pe_R1.fastq.trimmed  trimmed_fastq/pe_R2.fastq.trimmed -OP2 trimmed_fastq/mp_R1.fastq.int_trimmed trimmed_fastq/mp_R2.fastq.int_trimmed 2> scaffold.log
```
<img width="358" alt="Снимок экрана 2021-10-26 в 00 21 28" src="https://user-images.githubusercontent.com/93148512/138772896-d372e673-99ae-453f-91ec-baf48f386e35.png">

9. Создаем файл с одним самым большим размером:
```
echo scaffold1_len3834575_cov231 > name_scaff.txt
seqtk subseq Poil_scaffold.fa name_scaff.txt > BigScaff.fna
rm -r name_scaff.txt
```
10. Уменьшаем количество гэпов с помощью подрезанных чтений:
```
time platanus gap_close -o Poil -c Poil_scaffold.fa -IP1 trimmed_fastq/pe_R1.fastq.trimmed  trimmed_fastq/pe_R2.fastq.trimmed -OP2 trimmed_fastq/mp_R1.fastq.int_trimmed trimmed_fastq/mp_R2.fastq.int_trimmed 2> gapclose.log
```
11. Создаем файл с одним самым большим размером:
```
echo scaffold1_cov231 > name_scaff.txt
seqtk subseq Poil_gapClosed.fa name_scaff.txt > longest.fna
rm -r name_scaff.txt
```
Все необходимые расчёты приведены в Jupiter Notebook
