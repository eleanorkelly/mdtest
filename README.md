# How to debug the VA UI

## Prerequisites:
* Have a copy of [https://github.ibm.com/alchemy-va/utils](https://github.ibm.com/alchemy-va/utils). We will use the scripts in the bin directory.
* These scripts require some environment variables to be set - add these to `~/.bash_profile`:
  * VA_GIT_DIR - directory which contains the utils directory
  * BLUEMIX_USERID
  * PATH=$PATH:$VA_GIT_DIR/utils/bin - so that you can call the scripts from any directory
* Access to [https://hub.jazz.net/git/moosevan/vulnerability-advisor](https://hub.jazz.net/git/moosevan/vulnerability-advisor). 

## Debugging
1. Push an image to your bluemix account. You can use the custTest script to do this.

2. Log onto bluemix containers UI to check the status of the image.

3. For us, it said ‘incomplete’. 

4. Look at the VA_Workload dashboard in 'kibana 3' (see links below to kibana/graphana). Filter on your namespace - this should show the image going through all the VA backend stages. (You can get your namespace with the cmd `cf ic namespace get`.) Filter by adding a new filter where querystring=must and query="*NAMESPACE*". 
 * dev, prestage and stage logs and graphs are available at: [https://logmet.stage1.ng.bluemix.net/app/](https://logmet.stage1.ng.bluemix.net/app/)
 * prod-eu-gb logs and graphs are available at: [https://logmet.eu-gb.bluemix.net/app/](https://logmet.eu-gb.bluemix.net/app/)
 * prod-us-south logs and graphs are available at: [https://logmet.ng.bluemix.net/app/](https://logmet.ng.bluemix.net/app/)

5. There is a utils/bin/vaapi script which can talk to VA-API and query ES. Use this to check your images results in ES.

6. If this shows valid results in ES, check CCSAPI is working using utils/bin/vaesccsapiquery script. This should return an ID. HOW???

7. Go to the [alchemy dashboard](https://alchemy-prod.hursley.ibm.com/), then view, then logs (for the correct region), then infrastructure, this will open kibana (different to the one we looked at earlier). 

8. Change timeframe filter to last 6 hours. Query on ID from vaccsapiquery - Put `“ID”` in query text box at top. This shows all logs related to this image from CCSAPI. Select message field from left column. This shows messages - you should expect about 13 entries.

9. The first entry is the post that the CF app did. Last one is the result of the post. In the contents section of the post results there should be a list of results. This is an example of a result which gives incomplete: 

```
“Content={"hits": {"hits": [], "total": 0, "max_score": null}, "_shards": {"successful": 1150, "failed": 0, "total": 1150}, "took": 163, "nova": {"Image": "registry.ng.bluemix.net/brianpeacock/little-bad-image:20", "Id": "1de6d8045e2307f09612a1f4302575175587ef28af06c7cf1068465a5ac0b722", "Created": "1489498752"},”
```

10. The "hits" array is empty showing there are no results and "total: 0" confirms this. The “shards:successful:1150” and "failed =0" shows that ES has successfully queried all 1150 shards. This shows that the query to ES from CCSAPI is incorrect - as it is telling us ES knows no information on this image (but we know it does from step 5).

11. At the front of the message there is an id like this: `“req-20824-1489570443.979-2184524”`. Add this to your kibana search that you made in step 8 with ` OR “req-20824-1489570443.979-2184524”`. When it’s working you should get ---? This shows all of the entries from the registry, validation and CCSAPI to handle this request.

12. Look at the code for the CCSAPI: [https://github.ibm.com/alchemy-containers/api/blob/master/cloud/VizioCloudBackend.py](https://github.ibm.com/alchemy-containers/api/blob/master/cloud/VizioCloudBackend.py). The "validate_image" method shows the query to ES. For example, this is a query it made to ES: `url = "http://%s:9200/compliance-*/_search?pretty" % server`. The body of the curl is shown. This is called by [https://github.ibm.com/alchemy-containers/api/blob/master/api/v3/app.py](https://github.ibm.com/alchemy-containers/api/blob/master/api/v3/app.py)



_Note that rootkit annotator results are not seen by the UI.
