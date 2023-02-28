原文網址：https://gnomad.broadinstitute.org/news/2020-11-gnomad-v3-1-mitochondrial-dna-variants/

原文作者：Kristen Laricchia, Sarah E. Calvo

# Overview

gnomAD自v3.1版本開始提供粒線體DNA (mitochondrial DNA, mtDNA) 的變異資料。gnomAD團隊針對56,434個全基因組定序資料進行粒線體DNA的變異位點偵測，在最初始的釋出版本中共包含了10,850個獨特的變異位點 (unique variants)，這些變異涵蓋超過一半以上的粒線體基因組。在這56,434個全基因組定序資料中，共偵測到1,918,677個變異，其中高達98%的變異都是homoplasmic或接近homoplasmic variants，而2%屬於heteroplasmic variants。

粒線體DNA變異無論是對在許多人類疾病，抑或是人類演化遺傳學的研究都有其獨特的價值，希望可以藉由將粒線體DNA變異資料添加到gnomAD資料庫中，讓研究人員更完整地了解粒線體DNA變異與疾病的關聯。

在過去的版本中並沒有包含粒線體DNA變異位點的資料，是因為粒線體DNA的某些特性並不符合進行nuclear DNA變異位點偵測時所使用的假設，這些獨特的特性包含：

- 粒線體基因體為環型 (circular genome): 在進行reads alignment時，會使用線型的參考序列，但因為粒線體DNA為環狀結構，所以製作粒線體參考序列時會在原點製造出一個斷點。而在進行alignment時如果有橫跨在斷點上的reads就會造成拼裝上的問題。
- Heteroplasmy: 在一個細胞中會有數百到數千個粒線體DNA copies，多數的變異都是屬於homoplasmic (一個細胞內所有mtDNA copies都帶有一樣的變異)。然後也有些變異點是heteroplasmic (在一個細胞中只有部分的mtDNA帶有變異)，當這些heteroplasmic variants存在的比例很低時，就會造成偵測上的困難，且必須小心的區分這些variants是真的存在比例很低，抑或是因為technical artifacts或污染而造成。
- Nuclear sequences of mitochondrial origin (又稱作NUMTs): NUMTs是在nuclear genome中，源自於粒線體DNA的片段，原因是在演化的過程中，有粒線體的片段進入核，並嵌入nuclear genome中。有許多的NUMTs存在於參考序列中，然而，也有部分polymorphic NUMTs只存在於部分人當中，這些屬於NUMTs的reads很容易錯誤的拼在粒線體DNA上，並且造成low heteroplasmy的偽陽性結果。相反地，如果是粒線體的reads拼在參考序列上的NUMTs上，則會造成偽陰性的狀況。

為了解決上述的問題，gnomAD團隊設計了一個用於偵測粒線體變異的pipeline，並且使用這個pipeline做出了v3.1版本的mtDNA callset，這些資料包含了粒線體DNA的變異位點、heteroplasmic和homoplasmic variants的等位基因頻率 (allele frequencies) 等資料，並且預測這些variants會對下游的蛋白質、tRNA造成哪些影響。

# v3.1粒線體DNA callset

## 針對單一樣本的偵測pipeline

gnomAD團隊調整了GATK MuTect2，設計出專門偵測粒線體變異的模式，並使用GRCh38 chrM作為參考序列 (此參考序列等同於revised Cambridge Reference Sequence, rCRS, GenBankNC_012920.1)，此pipeline可以在[Terra平台](https://terra.bio/)上運行。

在設計pipeline時，遇到的問題及相應的解決方法如下所述：

- 環型的genome: 在粒線體DNA上可以分為兩個區域，分別是"control region"以及"non-control region"，control region橫跨在我們前面所說的斷點上 (座標為chrM:16024-16569 + chrM:1-576)。為了解決control region在偵測時所造成的困難，在進行alignment時，除了原本所使用的參考序列外，會另外準備一個參考序列是將斷點移動8000個鹼基，並使用這個新的參考序列進行control region的變異點偵測，最後再將control region上的變異位點座標轉回原處，再與non-control region上的變異位點合併。
- Heteroplasmic variants: nuclear DNA的變異使用同合子 (homozygous) 和異合子 (heterozygous) 來描述，而在[GATK MuTect2](#s1) variant caller中可以看到，所有粒線體DNA的變異會使用heteroplasmy level來描述。MuTect2的結果會在後續經由一連串的篩選標準選出可信度高的變異。
- Haplogroups: mtDNA並沒有重組 (recombination) 的過程，並且屬於母系遺傳。所以在研究粒線體的歷史與演化時，研究人員會把序列高度關聯的歸為同一個"haplogroup"。截至目前為止，共有超過5000個haplogroups，這些haplogroups與相應的基因型、演化過程都可以在[Phylotree資料庫](https://www.phylotree.org)中查詢。在分析流程中會使用[Haplogrep](https://github.com/seppinho/haplogrep-cmd)這個工具偵測個別樣本屬於哪個haplogroup。

需要特別留意的是這個分析流程只有在全基因定序的樣本中測試過，因為有許多exome capture的儀器會屏蔽粒線體的reads，導致最終粒線體DNA的覆蓋率並不足以準確的偵測出所有的變異點，尤其是low heteroplasmic variants。

## gnomAD Callset

為了得到粒線體DNA的callset，gnomAD團隊先針對每一個樣本進行變異位點的偵測，並且合併這些單一樣本的VCF，再使用嚴格的篩選標準選出可信度高的變異位點，篩選標準分別針對樣本、變異位點以及基因型來進行。以樣本的篩選標準為例，將mtDNA copies過高或過低，以及被污染的樣本進行排除。 而在本次的釋出中，亦將所有heteroplasmy < 10%的變異點排除，以避免這些訊號是從污染、定序錯誤或NUMT錯誤組裝而來的偽陽性結果，由於這是一個較嚴格的篩選條件，可能會誤篩選掉某些真正屬於低heteroplasmic variants，在後續的釋出中會再調整篩選條件，讓這些通過額外條件的變異可以被保留。

### Genome Variant Calls

上述討論的[變異位點偵測流程](https://portal.firecloud.org/?return=terra#methods/mitochondria/MitochondriaPipeline/25)是針對單一樣本來進行，並且使用Terra這個平台運行。針對覆蓋率 (coverage) 大於100X的homoplasmic calls會被歸到非變異位點的homoplasmic reference calls，如果覆蓋率小於等於100X的則會被認定為缺失的資料，並且不會納入最後計算等位基因頻率。

相較於nuclear DNA中，同合子與異合子在VCF中基因型會分別被標註成"1/1"與"0/1"，在粒線體DNA中，"1/1"代表homoplasmy (heteroplasmy level在95-100%之間)，"0/1"代表的是heteroplasmy (heteroplasmy level < 95%)。如果有小於1% heteroplasmic calls，那這個訊號會因為較高機率受定序儀器或其他因素干擾，而在分析過程中被移除。而在v3.1釋出中，另外將heteroplasmy level < 10%的變異移除 ("heteroplasmy_below_10_percent” genotype filter)，以確保所有的結果都具有較高的可信度。

# 補充說明

<p id="s1">1. MuTect2原先用作腫瘤中的變異偵測 (somatic variants)，因為粒線體的變異與腫瘤的特性有些許類似，同樣的位點，可能只有少部分的reads才帶有變異，所以gnomAD額外針對粒線體特性，包括很高的coverage、很低的heteroplasmy level，進一步調整參數，設計出mitochondria mode來專門偵測粒線體DNA的變異。</p>

# 相關資源

- [mtDNA變異資料下載](https://gnomad.broadinstitute.org/downloads#v3-mitochondrial-dna)