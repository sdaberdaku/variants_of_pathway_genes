# Variants of pathway genes

Scripts that collect the known variants of the genes in two human pathways from KEGG, insulin signaling (hsa04910) and tryptophan metabolism (hsa00380), from dbSNP, ClinVar and OMIM. They were written in February 2018 at the University of Padua. Genomic positions refer to GRCh38.p7.

## Contents

- The scripts, described below in the order they run.
- `insulin_signaling/` and `tryptophan_metabolism/`: the gene lists of each pathway and the results of the 2018 run for dbSNP and ClinVar: coding sequences (`*_CDS`, `*_CDS_RefSeq`), and per-protein VCF files and variant IDs (`*_dbSNP`, `*_ClinVar`).
- `variants_table.txt`: the final table of dbSNP and ClinVar variants.
- `clinvar.vcf.gz` and `variant_summary.txt.gz`: the ClinVar files used in 2018.

Not included:

- The OMIM files and everything derived from them. OMIM data is licensed to each user and cannot be redistributed; see [OMIM data](#omim-data).
- The KEGG KGML files. Step 1 downloads them again from the KEGG REST API.
- The full dbSNP VCF (`00-All.vcf.gz`, several GB). Step 6 explains how to download it.

## Requirements

- Python 2.7 with Biopython, PyVCF and requests
- [vcftools](https://vcftools.github.io/)

## OMIM data

Steps 9 to 11 need `OMIM/genemap2.txt`, the OMIM list of genes with their MIM numbers. To get it, register at https://www.omim.org/downloads and download `genemap2.txt` into a folder named `OMIM/` in the repository root. The other OMIM files are not used by the scripts. `.gitignore` excludes `OMIM/` and the OMIM results so they are not committed by mistake.

Step 9 downloads the allelic variants of each OMIM entry from omim.org. OMIM's terms of use apply. For programmatic access OMIM offers an [API](https://www.omim.org/api), which needs its own key.

Without the OMIM files, skip steps 9 to 11: `build_table.py` then builds the table from dbSNP and ClinVar only.

## Steps

Run the scripts from the repository root, in this order. They create the output folders they need.

1. `retrieve_and_parse_KGML_pathways.py` downloads the two pathways from KEGG and writes their genes to `insulin_signaling/insulin_signaling_genes.txt` and `tryptophan_metabolism/tryptophan_metabolism_genes.txt`.
2. `get_gene_names_and_symbols.py` looks up the official HGNC symbol and name of each gene and writes `official_gene_symbols_and_names.txt`. It appends the genes in `manually_added_genes.txt`, which come from published models of the two pathways (FBA, rule-based and dynamic models).
3. `fetch_all_cds.py` downloads the coding sequences of each gene.
4. `fetch_RefSeq_cds.py` downloads the coding sequences of each RefSeqGene.
5. `query_dbSNP_NCBI.py` queries dbSNP through Entrez for the coding non-synonymous SNPs of each gene.
6. `extract_dbSNP_VCF.sh` extracts those SNPs from the full dbSNP VCF. On the first run, uncomment line 27 to download `00-All.vcf.gz` (over 7 GB compressed, over 50 GB uncompressed). The script runs `N` extractions in parallel (line 29); set `N` to what your memory allows, for example 12 with 4 GB of RAM.
7. `parse_ClinVar_variants.py` downloads the ClinVar variant summary if `variant_summary.txt.gz` is missing, keeps the variants of the pathway genes and checks them against the coding sequence coordinates.
8. `extract_ClinVar_VCF.py` extracts the VCF records of the variants found in step 7 from `clinvar.vcf.gz`.
9. `fetch_OMIM_allelic_variants.py` downloads the OMIM allelic variants of the pathway genes (needs the OMIM files, see above).
10. `parse_OMIM_variants.py` keeps the OMIM variants of each gene and protein, as step 7 does for ClinVar.
11. `extract_OMIM_VCF.py` extracts the ClinVar VCF records of the OMIM variants found in step 10.
12. `build_table.py` merges everything into `variants_table.txt`.

## License

MIT, see [LICENSE](LICENSE). The license covers the scripts. The data files come from NCBI (dbSNP, ClinVar, RefSeq) and are subject to their terms.
