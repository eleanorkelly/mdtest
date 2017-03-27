---
layout: default
title: Guide for conductors on restarting the Logcrawler
type: PagerDuty
runbook-name: Restarting the Logcrawler service
description:
service: Vulnerability Advisor
failure: "Logcrawler lag on <hostname> is troubled"
ownership-details:
- owner: Vulnerability Advisor
  corehours: "UK"
  escalation: "Alchemy - Vulnerability Advisor"
---


## PD incident
For example **Logcrawler lag on prod-dal09-kraken1-host-21 is troubled**. [Link to example.](https://bluemix.pagerduty.com/incidents/P0ME98G)


## Actions to take

  * Get the hostname from PD alert title `Logcrawler lag on <hostname> is troubled`.
  * ssh onto the host
  * Run the command `sudo service alchemy-logcrawler restart` to restart the logcrawler.
  * Monitor the Executive Dashboard to make sure the service recovers as expected. See links below.
  
  
  - [Prod, Dallas](https://dashboard.rtp.raleigh.ibm.com/executive-dashboard#/default/va_prod_us_south/overview)
  - [Prod, London](https://dashboard.rtp.raleigh.ibm.com/executive-dashboard#/default/va_prod_eu_gb/overview)
  - [Stage, Dallas](https://dashboard.rtp.raleigh.ibm.com/executive-dashboard#/default/stage_dal09/overview)


  
## Escalation

  * If this doesn't resolve the lag problem after 5 minutes, escalate the pagerduty alert to "Alchemy - Vulnerability Advisor".
  * The pagerduty alert will not auto resolve - you will need to go to the exec dashboard to check the status of the check. 
  
## Further reading

  * [Another runbook with more information on resolving other crawler services.](https://alchemy-prod.hursley.ibm.com/docs/runbooks/pagerduty_crawlers.html)
