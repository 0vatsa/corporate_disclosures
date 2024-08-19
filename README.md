### What is this?
---
The following data dump contains raw PDF quarterly presentations and conference call transcripts of almost all companies listed on the NSE from Jan-2011 to Dec-2023.  
The dataset only contains quarterly investor presentations and conference call transcripts, if you also want the quarterly results and annual reports, ping me [here](mailto:srivatsarao@pm.me), I did not include them due to their size.<br><br>  
This was the data we used for writing a research paper wherein we developed a few models to evaluate the corporate disclosures. Similar to [1](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4098951 "1"), [2](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4490834 "2"), [3](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4425527 "3"), [4](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4835311 "4").  
The hardest part was getting this data because there are really no price-friendly commercial vendors with an API access. 

#### The entire dataset can be found here:
https://drive.google.com/drive/folders/1OKb3jaq743xG2H53y9A6H5GP1y2rMDO0?usp=drive_link 
### Usage 
---
- Everything is stored in SQLite.
- The data is split across 25 databases because the entire dataset is about **75GB**, so I had to split them for easier transfer.
- In the drive there is a file `each_company_location.json`, this contains data as to which company is in which of the 25 databases. 

```json
{ 
	"1": ["20 Microns",  "21st Cent Mgt", "360 One Wam", ... ],
	"2": ["Bhagiradha Chem", "Raj Packaging Inds", "Imagicaaworld Enter", ...],
	.
	.
	.
	"25":  ["Utkarsh Small Fin.", "Uttam Sugar Mills", "V2 Retail", ...]
}
```
- There is also a `missing_data.json`, this contains which company's data is **missing** for which year and quarter. 

```json
{
	"20 Microns": {  
		"2011": {
			"Conference Call": [ "Q1", "Q2", "Q3","Q4"],  //implies in 2011 all quarters con call transcript is missing
			"Investor Presentation": ["Q1", "Q2", "Q3","Q4"],
			"Earnings Release": ["Q1", "Q2", "Q3","Q4"],
			"Annual Report": []      //if empty implies annual report is NOT missing
		}
		"2012": {...},
		.
		.
		"2023": {..}
	},
	"21st Cent Mgt": {...},
	.
	.
	.
}
```
- Here is the schema:

| column  | description  |
| :------------ | :------------ |
| `id` | primary key of the database  |
| `year`  | the year the PDF (corporate disclosure) was published in |
| `quarter`  | which quarter of the year, can take the values `Q1, Q2, Q3, Q4`  |
| `filing_type` | whether the PDF is an investor presentation or conference call transcript, can take the values `Conference Call` or `Investor Presentation` |
| `y_q_id` | a unique id to identify the document, format: `(company name)_(year)_(quarter)_(filing type)` example: `ABB India_2023_Q2_CC` |
| `t_id`  | redundant, not required  |
| `t_short_name`  | Company name  |
| `b_file`  | actual raw binary PDF |

---
- Sample in-memory processing of the PDF, this is the intended way to use the database:   

```python
import io
import sqlite3
import pandas as pd

connection_number = 1 #the database number

conn = sqlite3.connect('corporate_filings_split_number_{}.db'.format(connection_number)) #change location to wherever you have the database
sql_query = "SELECT * FROM corporate_filings"
chunks = pd.read_sql(sql_query, conn, chunksize=25)

for chunk in chunks:
    print(chunk)
    
    if not chunk.empty:
        
        for i in range(len(chunk)):
            
            pdf_binary = io.BytesIO(chunk['b_file'].iloc[i]) 
			#load as io bytes and pass to pymupdf or PyPDF2
            
            #process whatever 
                 
            break 
    break 
                   
conn.close()

```


- Sample PDF extraction from the database to file:  

```python
import sqlite3
import pandas as pd

connection_number = 1 #the database number

conn = sqlite3.connect('corporate_filings_split_number_{}.db'.format(connection_number)) #change location to wherever you have the database
sql_query = "SELECT * FROM corporate_filings"
chunks = pd.read_sql(sql_query, conn, chunksize=25)

for chunk in chunks:
    
    print(chunk)
    
    if not chunk.empty:
        
        for i in range(len(chunk)):
            
            with open(r'extracted_pdf_{}.pdf'.format(chunk['y_q_id'][i]), 'wb') as pdf:
                pdf.write(chunk['b_file'][i])
                
            break 
    break 

conn.close()
```
