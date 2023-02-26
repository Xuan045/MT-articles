原文網址：https://gnomad.broadinstitute.org/news/2020-11-gnomad-v3-1-mitochondrial-dna-variants/

原文作者：Kristen Laricchia, Sarah E. Calvo

# Overview

gnomAD自v3.1版本開始提供粒線體DNA (mitochondrial DNA, mtDNA) 的變異資料。gnomAD團隊針對56,434個全基因組定序資料進行粒線體DNA的變異位點偵測，在最初始的釋出版本中共包含了10,850個獨特的變異位點 (unique variants)，在粒線體基因體中超過一半的鹼基對都被發現有變異點存在。在這56,434個全基因組定序資料中，共偵測到1,918,677個變異，其中高達98%的變異都是homoplasmic或接近homoplasmic variants，而2%屬於heteroplasmic variants。

粒線體DNA變異無論是對許多人類疾病，抑或是人類演化遺傳學的研究都有其獨特的價值，希望可以藉由將粒線體DNA變異資料添加到gnomAD資料庫中，讓研究人員更完整地了解粒線體DNA變異與疾病的關聯。

在過去的版本中並沒有包含粒線體DNA變異位點的資料，是因為粒線體DNA的某些特性並不符合進行nuclear DNA變異位點偵測時所使用的假設，這些獨特的特性包含：

- 粒線體基因體為環型 (circular genome)：在進行reads alignment時，會使用線型的參考序列，但因為粒線體DNA為環狀結構，所以製作粒線體參考序列時會在原點製造出一個斷點。而在進行alignment時如果有橫跨在斷點上的reads就會造成問題。
- Heteroplasmy: 在一個細胞中會有數百到數千個粒線體DNA copies，多數的變異都是屬於homoplasmic (一個細胞內所有mtDNA copies都帶有一樣的變異)。然後也有些變異點是heteroplasmic (在一個細胞中只有部分的mtDNA帶有變異)，當這些heteroplasmic variants存在的比例很低時，就會造成偵測上的困難，且必須小心的區分這些variants是真的存在比例很低，抑或是因為technical artifacts或污染而造成。
- Nuclear sequences of mitochondrial origin, NUMTs: NUMTs是在nuclear genome中，源自於粒線體DNA的片段，原因是在演化的過程中，有粒線體的片段進入核，並嵌入nuclear genome中。有許多的NUMTs存在於參考序列中，然而，也有部分polymorphic NUMTs只存在於部分人當中，這些屬於NUMTs的reads很容易錯誤的拼在粒線體DNA上，並且造成low heteroplasmy的偽陽性結果。相反地，如果是粒線體的reads拼在參考序列上的NUMTs上，則會造成偽陰性的狀況。

為了解決上述的問題，gnomAD團隊設計了一個用於粒線體變異的偵測pipeline，並且使用這個pipeline做出了v3.1版本的mtDNA callset，這些資料包含了粒線體DNA的變異位點、heteroplasmic和homoplasmic variants的allele frequencies等資料，並且預測這些variants會對下游的proteins, tRNA造成哪些影響。


# 相關資源

- [mtDNA變異資料下載](https://gnomad.broadinstitute.org/downloads#v3-mitochondrial-dna)