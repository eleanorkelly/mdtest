# How to debug the VA UI

## Prerequisites:
* Have a copy of [https://github.ibm.com/alchemy-va/utils](https://github.ibm.com/alchemy-va/utils). We will use the scripts in the bin directory.
* Access to [https://hub.jazz.net/git/moosevan/vulnerability-advisor](https://hub.jazz.net/git/moosevan/vulnerability-advisor). 

## Debugging
- Push an image to your bluemix account. You can use the custTest script to do this.

- Log onto bluemix containers UI to check the status of the image.

- For us, it said ‘incomplete’. 

- Look at kibana. This is linked to from the [prod dashboard](https://alchemy-prod.hursley.ibm.com/), go to 'view' then 'system events' for the correct region (where you pushed your image). Look at the VA_Workload dashboard. Filter on your namespace - this should show the image going through all the VA backend stages.

- There is a utils/bin/vaapi script which can talk to VA-API and query ES. Use this to check your images results in ES.

- If this shows valid results in ES, check CCSAPI is working using utils/bin/vaesccsapiquery script. This should return an ID. HOW???

- Go to the alchemy [dashboard](https://alchemy-prod.hursley.ibm.com/), then view, then logs (for the correct region), then infrastructure, this will open kibana (different to the one we looked at earlier). 

- Change timeframe filter to last 6 hours. Query on ID from vaccsapiquery - Put “ID” (in speech marks) in query text box at top. This shows all logs from things CCSAPI has done with this image. Select message field from left column. This shows messages - you should expect about 13 entries. 

- The first entry is the post that the CF app did. Last one is the result of the post. In the contents section of the post results there should be a list of results. This is an example of a result which gives incomplete 


“Content={"hits": {"hits": [], "total": 0, "max_score": null}, "_shards": {"successful": 1150, "failed": 0, "total": 1150}, "took": 163, "nova": {"Image": "registry.ng.bluemix.net/brianpeacock/little-bad-image:20", "Id": "1de6d8045e2307f09612a1f4302575175587ef28af06c7cf1068465a5ac0b722", "Created": "1489498752"},”

- The hits array is empty showing there are no results and ‘total 0’ confirms this. The “shards:successful:1150” Shows that ES has successfully queried all 1150 shards (failed =0). This shows that the query to ES from CCSAPI is incorrect - ES knows no information on this image or the query is incorrect. 

- At the front of the message there is an id like this: “req-20824-1489570443.979-2184524”. Search for this in kibana. with ` OR “req-20824-1489570443.979-2184524” `. When it’s working you should get _? . They are all of the entries from the registry, validation and CCSAPI to handle this request. 
- Looking at the code for the CCSAPI (https://github.ibm.com/alchemy-containers/api/blob/master/cloud/VizioCloudBackend.py) 
- Look at validate_image method…it shows the query to ES. For example, this is a query it made to ES : “url = "http://%s:9200/compliance-*/_search?pretty" % server”. The body of the curl is shown.
- (This is called by https://github.ibm.com/alchemy-containers/api/blob/master/api/v3/app.py )
- Use script es_ccsapi_query ? (get correct name). 
