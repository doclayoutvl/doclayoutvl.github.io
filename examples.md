Example A: Layout as Segmentation
Input Prompt: "Generate mask-based layout segmentation of all semantic regions."
Model Output:

article_title [LAY]
article_subtitle [LAY]
section-header [LAY]
section-text [LAY]
image_region [LAY]
section-header [LAY]
section-text [LAY]
image_region [LAY]
section-header [LAY]
section-text [LAY]
image_region [LAY]
page_number [LAY]

Example B1: Open-Set Layout Segmentation (Generic Ontology)
Input Prompt: "Segment the document into the following layout classes: picture, header, title, section header, table, table header"

Model Output:

header [LAY]
picture [LAY]
picture [LAY]
title [LAY]
table [LAY]
section header [LAY]
table header [LAY]
table [LAY]
table header [LAY]
table [LAY]
table header [LAY]
table [LAY]

Example B2: Open-Set Layout Segmentation (Domain-Specific Ontology)

Input Prompt: "Segment the document into the following insurance-specific layout classes: bank logo, certification logo, insurer information block, schedule tile, policyholder details, covered members table, premium breakdown, legal clause, nominee information, regulatory info block, authorization signature"

bank logo [LAY]
certification logo [LAY]
insurer information block [LAY]
schedule tile [LAY]
policyholder details [LAY]
nominee information [LAY]
covered members table [LAY]
premium breakdown [LAY]
regulatory info block [LAY]
authorization signature [LAY]
legal clause [LAY]

Example C: Hierarchical Layout Segmentation

Input Prompt: "Semantically and hierarchically segment this mutual fund factsheet into a layout with exactly three top-level regions: header, body, and footer, so that a new investor can quickly grasp the document's structure."

Model Output:

<document>
  <header> 
    fund-title [LAY] 
    sub-title [LAY] 
    issue-data-strip [LAY] 
    logo [LAY] 
  </header> [LAY]
  <body>
    fund-info-table [LAY]
    <performance-section>
      old-line-chart [LAY] 
      new-line-chart [LAY] 
      source [LAY]
    </performance-section> [LAY]
    asset-allocation-section [LAY]
  </body> [LAY]
  <footer> 
    page-number [LAY] 
  </footer> [LAY]
</document>

Example D1: Promptable Layout Understanding (Targeted Extraction)

Input Prompt: "Mask all personal identifiable information of the passport holder"

Model Output:

photo [LAY]
signature [LAY]
passport_num [LAY]
surname [LAY]
given_name [LAY]
sex [LAY]
date_of_birth [LAY]
nationality [LAY]
place_of_birth [LAY]
place_of_issue [LAY]
date_of_issue [LAY]
date_of_expiry [LAY]
mrz_block [LAY]

Example D2: Promptable Layout Understanding (Spatial Reasoning)

Input Prompt: "Find the figure to the left of a table"

Model Output:

This document utilizes a two-column format containing a total of two figures and two tables. Specifically, Figure 2 ("The installed instrumentation") is the only figure positioned to the left of a table, which is Table 1 ("Accelerations measured on the inner frame").

Final Answer: Figure 2 [LAY]