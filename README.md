 # Accelidea POC

 ## Quickstart
 To start the app, first get the `.env` file from me, then, if you have docker installed just run:
 ```
docker build -t accelidea_poc .
docker run --env-file .env -p 8000:8000 accelidea_poc
```
you should then be able to access the app at [http://127.0.0.1:8000/upload](http://127.0.0.1:8000/upload).

## App Description
### Overview
This app takes multiple pdfs and processes them to extract information, it tracks what information came from what uploaded document and saves a copy of the uploaded documents with a unique id for future reference. It also detects if a document has been uploaded before via an md5 hash so it does not reprocess the same document twice. 
The extraction of fields tries a dumb regex approach and, if that fails, makes a call to Claude to try to get the value, it then logs any found values into a table for audit-ability. The OCR extracted text is also saved in the database so we can display any and all additional fields outside our data model if needed.
After the input data has been transformed to fit our data model, we get the three output documents and populate them with the values, zip them up and give them to the user. 
### Technology Choices
I used sqlite and fastAPI to get things moving quickly, I think in a proper implementation I would use cloud services like RDS and I would weigh my choice of framework more carefully to make sure it fit the use case. Because the AI is the bottleneck it would be useful to optimize everything else that we can control for user experience. 
### What was left out
#### UI
The UI is extremely basic, I focused on the core capabilities, this is a reflection of both my shortcomings (I'm more of a backend developer) and the time constraints. Also, the UI required to put a copy of the artifact next to the associated extracted fields was not built but that data is on the system and that could be built on top of this. 
#### Completeness Percents
The completeness percents can be derived from the extracted data but are not displayed to the user, I focused more on backend plumbing here and not on UI.
#### Suggested Values for Empty Fields
This was skipped. The reason is we have to be very careful about not mixing the suggestions with values that derived from the actual source documents. This means bundling up the call to the LLM would not work and we need a separate call. Another reason to hold off on this is that we can use other uploaded documents to train a model to give us good defaults instead of having the LLM just guess them out of the gate, which will give a poor user experience. 
## Other Notes
The way this app is structured we don't need to determine a "document type" we just take all incoming information. The LLM can handle that and I think having any expectation as far as what documents may be coming in would make the program more brittle, and we need flexibility. 
Also I may need to talk more to Rose here but when trying to extract new fields, i.e. "self-evolving" I thought the best approach would be to keep the OCR outputs and run periodic jobs on them to determine similarities. I would probably keep these fields in JSON column in the db and "promote" them to proper data model fields as needed. The reason one might want to have be proper fields in the data model is that would assist with searchability which would make for better reccomendations going forward. 
Linking of fields is solved via the data model. So if a field is linked to another field, we would just use the first field for all instances of the second field. This may need to be refined. So, for example, if a field is not identical to another field but derivate of it, we may way to account for that but given the time constraint I did not tackle this. 
## What is AI vs Deterministic
This implementation is heavily AI because it does not rely on static document types but takes any document and tries to fit it in to the data model. However there are some important features that are not AI, first the deduplication is deterministic and second the app does try to get some of the data model attributes via RegEx. The reason for this is both cost and speed, AI is expensive and time consuming, if there are simpler, faster, and cheaper options they should be tried first. 
## Scalability
The chief concern I have about this implementation is the slowness and cost of the AI. I think I could scale this pretty well by using smaller servers and high levels of parallelization but from a business perspective we would need to optimize for AI spend above all else. 
