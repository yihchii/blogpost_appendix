## Appendix: HG001 WES Quality Reiview 


While working on the secondary variant calling pipeline benchmark (+link TBD), we found that HG001 WES has significant lower accuracy, in both recall and precision, comparing to other samples (HG002&ndash;5) (Figure 1.). 
<p align="center">
<img src="https://github.com/yihchii/blogpost_appendix/blob/e3a5472a3650eea9b6c304fd4d0b0eb0a550ef61/BY_SAMPLE_WES_SNP_RecallPrecision.png" alt="Figure1." height="477.75" width="455.25" />
<br>
<em> <b>Figure 1.</b> Accuracy metrics for SNPs called for each WES sample. The top ends of the bars indicate observed mean values, and the error bars represent the 95% confidence intervals around the means. Top: false negative rate (FNR); bottom: false discovery rate (FDR). Smaller values of FNR and FDR (x-axes) demonstrate higher accuracy. </em>
</p>



To understand the cause of such discrepancy, we looked further into how the libraries were prepared and the quality metrics of all five NIST WES samples. We first noticed that the capture protocol used for HG001 is different from HG002&ndash;5 (Table 1.). This can be one of the major reasons on the different accuracy discoveries. 



**Table 1.** Description of the data source and exome capture libraries for all 5 NIST WES samples (HG001&ndash;5).
| WES library        | HG001 WES                                                    | HG002&ndash;5 WES                                                  |
| :----------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| FASTQ  Source      | [GIAB FTP site](http://ftp-trace.ncbi.nih.gov/ReferenceSamples/giab/data/NA12878/Garvan_NA12878_HG001_HiSeq_Exome/) | HG002: [SRR2962669](https://www.ncbi.nlm.nih.gov/sra?term=SRR2962669); HG003: [SRR2962692](https://www.ncbi.nlm.nih.gov/sra?term=SRR2962692);<br> HG004: [SRR2962694](https://www.ncbi.nlm.nih.gov/sra?term=SRR2962694); HG005: [SRR2962693](https://www.ncbi.nlm.nih.gov/sra?term=SRR2962693) |
| Exome captured kit | [Nextera Rapid Capture Exome and Expanded Exome <br>Enrichment Kits](https://support.illumina.com/content/dam/illumina-marketing/documents/products/datasheets/datasheet_nextera_rapid_capture_exome.pdf) | [Agilent SureSelect Human All Exon V5 kit](https://www.agilent.com/cs/library/datasheets/public/AllExondatasheet-5990-9857EN.pdf) |
| Target content     | Exons, UTRs, and miRNA                                       | Exons                                                        |
| \# Taget exons     | 201,121                                                      | 357,999                                                      |



We then used [Qualimap v2.2.1](http://qualimap.bioinfo.cipf.es/) ([DNAnexus app](https://platform.dnanexus.com/app/app-F9FBZgj9bgKVx8YxF8zjq804)) to assess at the quality control statistics of each of the WES samples. 

Command used in `qualimap` analysis:

```bash
qualimap bamqc -bam markdup.bam \
               --feature-file target_region.bed \
               -outdir /home/dnanexus/qoutput \
               -outformat HTML \
               -outfile sample.bamqc.html
```


**Table 2.** Quality control metrics for all five NIST WES samples collected from Qualimap report.
| Sample                                 | HG001       | HG002       | HG003       | HG004       | HG005       |
| -------------------------------------- | ----------- | ----------- | ----------- | ----------- | ----------- |
| Number of reads                        | 165,650,726 | 150,538,907 | 128,737,272 | 146,314,721 | 122,117,967 |
| Average read length                    | 96.67       | 125.92      | 125.92      | 125.91      | 125.92      |
| % Mapped Reads                         | 99.97%      | 99.72%      | 99.74%      | 99.74%      | 99.76%      |
| Average coverage (inside of regions)   | 141.1X      | 238.6X      | 201.7X      | 225.8X      | 193.4X      |
| % Mapped Reads (inside of regions)     | 65.45%      | 77.46%      | 76.83%      | 75.36%      | 77.6%       |
| % Duplicated reads (inside of regions) | 15.38%      | 5.07%       | 4.37%       | 5.16%       | 5.86%       |


We first noticed that while HG001 has abundant number of reads (165.65 million), the on-target coverage is relatively low (141X) comparing to other samples (~200X) (Table 2.). Also, the reads mapped inside of the capture regions are significantly lower than other samples (66.45% vs. >75%). We suspected that the quality of this HG001 WES library might be low. This may be caused by the capture regions of HG001 WES includes UTR regions, which is more challenged to capture and align when the sequences are considered low complexity. 

To check if our hypothesis is correct, we reprocessed the HG001 WES sample with all pipelines and used only the exome region ("[Nextera Rapid Capture Exome vs. Expanded Exome Overlap File](https://support.illumina.com/softwaredownload.html?assetId=8f1948ce-34e1-4105-aa05-8fa69f3c009a&assetDetails=nexterarapidcapture_exome_vs_expandedexome_overlap.bed)") as the target file instead of the original, expended captured region. In Figure 2., we observed great improvements with respect to both recall and precision when we excluded low-complexity UTRs for variant calling. However, the accuracy metrics remain less preferred comparing to the other samples.

<p align="center">
<img src="https://github.com/yihchii/blogpost_appendix/blob/e3a5472a3650eea9b6c304fd4d0b0eb0a550ef61/HG001_WES_SNP_RecallPrecision.png" alt="Figure2." height="251.25" width="624.75"  />
<br>
<font size="-10">
<em> <b>Figure 2.</b> Accuracy metrics for SNPs called for HG001 WES sample with original capture regions (HG001) and within exome regions (HG001ExpandedOverlapExome) comparing to other samples (HG002). The top ends of the bars indicate observed mean values, and the error bars represent the 95% confidence intervals around the means. Top: false negative rate (FNR); bottom: false discovery rate (FDR). Smaller values of FNR and FDR (x-axes) demonstrate higher accuracy. </em>
</font>
</p>


We also found that HG001 has much higher on-target duplicated read percentage comparing to other samples (15.38% vs. 4.37&ndash;5.86%). This might suggest the exome-sequencing was started with lower DNA materials, which could result in PCR over-amplify the insufficient amount of DNA molecules. 

It is challenging for us to trace back and pinpoint what exactly caused the low quality of the WES sample, especially since this data is seven years old (released in 2013). Because the metrics are suggesting the sample doesn't come with good quality, and we couldn't effectively fix the quality post-sequencing, we decided to drop this sample from our report. We look forward for more data generated for GIAB samples to come.

